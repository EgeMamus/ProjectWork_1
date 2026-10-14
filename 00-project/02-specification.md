---
project_outline: "[[01-project-outline]]"
project_id: "2026-OG-04"
supervisor: "Gábor OLÁH"
status: "Proposed architecture and requirements; subject to validation"
---

# Optimized Planning of Personalized Running and Cycling Loops Based on OpenStreetMap Data

## 1. Actors and permissions

| Role                   | Needs                                                | Responsibility / access                                                                 |
| ---------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Runner                 | Find a convenient loop matching distance and effort  | Select start, distance, surface, terrain and ascent preferences; inspect proposed loops |
| Cyclist                | Find a cycling-permitted loop with suitable surfaces | Choose cycling mode and preferences; inspect viable loops                               |
| Evaluator / supervisor | Verify correctness and compare algorithmic methods   | Run repeatable workloads; inspect quality, runtime, scalability and tests               |
| Maintainer / developer | Prepare data and operate the system                  | Update OSM/elevation datasets and routing rules; run tests and deploy the application   |

The first version does not require user accounts. Maintenance interfaces and internal datasets are not exposed to anonymous users.

## 2. Use cases / user stories

### UC1: Configure a loop request

- **Actor:** Runner or cyclist.
- **Precondition:** The map and supported test area are available.
- **Main flow:** (1) Select an activity mode. (2) Pick a starting location on the map. (3) Enter the target distance and optional tolerance. (4) Set ascent, surface and terrain preferences, and optional hard exclusions. (5) Request route generation.
- **Alternative/error flows:** Reject invalid distance; explain a start outside supported map coverage; snap the selected location to a compatible graph location and communicate this adjustment.
- **Postcondition:** A validated routing request is passed to the optimizer.

### UC2: Generate personalized closed-loop candidates

- **Actor:** Runner or cyclist.
- **Precondition:** A valid request and traversable OSM network exist.
- **Main flow:** (1) Filter the network according to activity access. (2) Choose candidate waypoint sequences. (3) Connect waypoints through permitted graph paths, returning to the start. (4) Verify closure and compute statistics. (5) Enforce hard constraints and rank feasible routes. (6) Return one or more distinct candidates.
- **Alternative/error flows:** If no acceptable loop is found within search and time bounds, return an explicit no-route result with possible relaxations; never display an invalid path as a valid loop.
- **Postcondition:** Ranked loop candidates and their metrics are available.

### UC3: Compare route options on a map

- **Actor:** Runner or cyclist.
- **Precondition:** The route engine returned candidates.
- **Main flow:** (1) Show the candidate paths on a map. (2) Display length, distance deviation, ascent if known, surface distribution including unknown values, and a route-quality summary. (3) Select a route to highlight it.
- **Alternative/error flows:** Show missing elevation/surface data transparently and avoid reporting fabricated ascent values.
- **Postcondition:** The user understands route trade-offs.

### UC4: Compare algorithms with controlled experiments

- **Actor:** Evaluator.
- **Precondition:** The same frozen OSM graph, evaluation rules and set of requests are available.
- **Main flow:** (1) Run baseline waypoint + A\*/Dijkstra routing. (2) Run genetic waypoint optimization. (3) Validate each output. (4) Record feasibility, route quality, runtime and memory; repeat stochastic experiments with recorded random seeds. (5) Analyze trends as test areas increase in size.
- **Alternative/error flows:** Record timeouts or no-route outcomes rather than excluding them.
- **Postcondition:** Reproducible comparison data and analysis.

### UC5: Handle gaps in source data

- **Actor:** System (with evaluator review).
- **Precondition:** OSM edges lack surface/terrain tags or external elevation is unavailable.
- **Main flow:** (1) Mark data as missing. (2) Apply an explicit and documented missing-data policy. (3) Expose uncertainty in statistics and optionally deprioritize poorly described routes.
- **Postcondition:** The system does not equate missing information with asphalt, flat terrain or unrestricted access.

## 3. Functional requirements

| ID   | Requirement                                             | Acceptance criteria                                                                                                                             | Priority |
| ---- | ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| FR1  | Import an OSM-based routable network.                   | Given a chosen test area, a graph with nodes, directed edges, lengths and available access/surface metadata is constructed and loadable.        | Must     |
| FR2  | Distinguish running and cycling access.                 | Routing never uses edges classified as prohibited for the chosen activity under the configured tagging policy.                                  | Must     |
| FR3  | Accept start, target distance and preferences.          | Valid input is accepted; negative/zero distances and unavailable locations receive clear errors.                                                | Must     |
| FR4  | Generate closed-loop routes.                            | A returned route starts and ends at the same snapped graph node and every successive edge is traversable and mode-compatible.                   | Must     |
| FR5  | Support distance/elevation/surface/terrain preferences. | Modifying each preference changes filtering or scoring in a documented manner on a controlled fixture graph.                                    | Must     |
| FR6  | Calculate route metrics.                                | Returned routes include network length, ascent when data supports it, and known/unknown surface proportions; derived totals pass fixture tests. | Must     |
| FR7  | Display route(s) on an interactive map.                 | User can see loop geometry and read its route metrics and preferences.                                                                          | Must     |
| FR8  | Implement a reproducible baseline.                      | Waypoint candidate generation plus A\* or Dijkstra path stitching generates a valid loop on a prepared fixture network.                         | Must     |
| FR9  | Implement a distinct optimization approach.             | A genetic waypoint search (current proposal) generates valid candidates and uses the common evaluator.                                          | Must     |
| FR10 | Benchmark both approaches.                              | A repeatable experiment produces machine-readable quality, runtime, success and scale measurements on identical requests.                       | Must     |
| FR11 | Provide automated algorithm tests.                      | Graph validity, closure, scoring, input constraints and route metrics are covered by pytest tests.                                              | Must     |
| FR12 | Expose a backend routing interface.                     | A REST request returns route coordinates, metrics, preferences, status and algorithm identifier.                                                | Should   |
| FR13 | Return multiple distinct candidate routes.              | When available, the system presents alternatives and avoids near-duplicate geometry.                                                            | Should   |
| FR14 | Cache and persist prepared spatial data.                | Repeated regional requests can reuse prepared graphs; optional PostgreSQL/PostGIS persistence can be added without replacing algorithm code.    | Should   |

**Priority definitions:** Must = required for assignment completion; Should = desirable product-level improvement; Could = future extension.

## 4. Business rules, data model and algorithm design

### 4.1 Graph representation and access

- Use an attributed **directed graph** `G=(V,E)`; nodes represent OSM road/path junctions and edges represent traversable directed segments. Preserve parallel edges if they encode distinct traversals, access or geometries.
- Edge attributes: length in meters; geometry; activity access; surface and terrain/way type tags where supplied; source identifiers; elevation gain/loss after suitable elevation data is attached.
- A route is an ordered sequence of valid directed edges. **Hard invariants:** no disconnected jumps; legal travel direction; chosen-mode access; first and last node equal; explicit limits for prohibited surfaces/terrain.
- Separate preprocessing (OSMnx/OSM metadata and external elevation), route generation, route scoring and UI serialization.

### 4.2 Baseline: waypoint generation + shortest paths

1. Snap the user-provided start to a suitable traversable node.
2. Sample candidate waypoint sequences in different bearings and distance bands, to encourage loops rather than immediate return on the same path.
3. Connect consecutive waypoints and the start via A* or Dijkstra on legal edges. Distances or other nonnegative edge costs provide path weights; heuristic admissibility must be respected if optimality is claimed for A*.
4. Concatenate candidate paths, validate closed-loop feasibility, calculate repeated-edge proportion and other penalties.
5. Reject violations, rank remaining routes by the shared cost model and diversify the returned set.

Dijkstra/A\* find point-to-point paths; loop generation comes from the waypoint search and path stitching. No claim of globally optimal loops is made.

### 4.3 Alternative: genetic waypoint optimization (planned)

- Encode individuals as waypoint sequences or constrained waypoint parameters, not arbitrary unconnected edge sequences.
- Initialize from feasible geographic candidates. Apply selection, crossover and mutation to the waypoint representation; reconstruct and validate each candidate using the same graph and shortest-path routine.
- Score with the **same route evaluator** as the baseline, penalizing or rejecting invalid individuals; retain elites if experiments justify it.
- Record population size, iterations, initialization method, random seed and runtime budget. Tune hyperparameters on a separate development workload; benchmark on fixed unseen test requests.
- **Fallback research option:** Simulated Annealing on the same waypoint representation if genetic search proves impractical. At least two distinct methods must still be implemented and compared.

### 4.4 Multi-criteria objective and constraints

For feasible route `R`, the initial proposed cost to **minimize** is:

`C(R) = w_d * D(R) + w_e * E(R) + w_s * S(R) + w_t * T(R) + w_r * P(R)`

- `D(R)`: normalized absolute deviation of route length from requested distance.
- `E(R)`: normalized undesirable elevation gain relative to user-selected ascent tolerance or preference (not necessarily penalized when the user seeks hills).
- `S(R)`: normalized share of known disfavored surfaces, with an explicit separate handling policy for unknown tags.
- `T(R)`: normalized terrain-type mismatch.
- `P(R)`: share or length of repeated traversals/segments, consistently defined across algorithms.
- `w_*`: nonnegative, user-adjustable or preset profile weights (normalized to sum to 1 for interpretability).

Absolute constraints such as valid closure, activity access, forbidden terrain/surfaces and any specified maximum distance deviation are checked before ranking. Scales, treatment of unknown data and weight presets are provisional and require sensitivity testing. Report raw metrics alongside the aggregate score.

### 4.5 Data and reproducibility

- OSMnx prepares and inspects networks; NetworkX supports shortest paths and test graph operations. Elevation is attached from a separately selected licensed source where required; OSM road data alone does not guarantee height data.
- Record data extract date, geographic boundary, activity filters, code revision, algorithm settings and random seeds. Follow source attribution and usage policies.
- Never commit tokens, secret keys, private user tracks or production data to Git.

## 5. User interface, system architecture and workflow

### Planned technology choices (provisional)

| Layer               | Proposed technology     | Rationale                                                   |
| ------------------- | ----------------------- | ----------------------------------------------------------- |
| Route/data research | Python, OSMnx, NetworkX | Rapid graph-based prototyping and algorithm experimentation |
| Backend             | FastAPI                 | Typed REST endpoints; separate engine from UI               |
| Frontend            | React + TypeScript      | Maintainable web interface and typed client state           |
| Map                 | MapLibre GL JS          | Interactive mapping and polyline display                    |
| Persistence (later) | PostgreSQL + PostGIS    | Spatial storage and querying once justified by data volume  |
| Automated tests     | pytest                  | Unit and integration tests of core algorithms               |
| Versioning          | Git/GitHub              | Revision history and reproducible collaboration             |

**Logical flow:** OSM / optional elevation data → preprocessing / activity-specific graph → route generators → shared route validator and scorer → FastAPI JSON/GeoJSON response → React/MapLibre map.

**Main screens:** map and starting-point selector; mode/distance/preference controls; loading and no-route feedback; candidate route list with statistics and map highlighting. Accounts and live tracking are not included initially.

**Suggested API contract:** `POST /routes/generate` with mode, start coordinates, target distance, tolerance, surface/terrain/ascent preferences and optionally algorithm; returns status, snapped start, candidate GeoJSON geometries, length, ascent availability, surface shares, quality metrics and generator identifier. Exact schema will be frozen during implementation.

## 6. Non-functional requirements, testing and evaluation

| Requirement / research measure | Verification approach                                                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Correctness                    | Unit tests on small directed fixture graphs (including one-way edges, disconnected regions, missing tags and impossible requests); every delivered route validated for closure and access. |
| Distance fidelity              | Report absolute/relative target distance error and distribution across all requests; any tolerance is user-specified or explicitly configured.                                             |
| Route quality                  | Report ascent, preferred surface share, terrain mismatch, repeated-edge share and normalized combined cost, not only a single score.                                                       |
| Runtime                        | Record end-to-end generation latency and algorithm-only time, including median and tail values across comparable runs.                                                                     |
| Scalability                    | Run increasing region sizes / node and edge counts while holding requests, compute budget and dataset conditions as comparable as possible.                                                |
| Reliability                    | Report valid-loop rate, timeouts and no-route outcomes; never discard failures from averages without disclosure.                                                                           |
| Reproducibility                | Freeze prepared graph versions, record random seeds, configuration and timing hardware; repeat stochastic algorithms multiple times.                                                       |
| Maintainability                | Keep data preparation, optimization, scoring, API and visualization in separate modules.                                                                                                   |
| Data quality                   | Surface unknown OSM tags and missing elevation; explain fallback rather than inventing data.                                                                                               |
| Security / repository hygiene  | No keys, access tokens, private tracks, personal information or production data in the repository.                                                                                         |

**Experiment design:** Use a fixed set of running/cycling requests spanning distances (e.g. 5, 10, 15 and 20 km), elevation/surface preferences and test areas of different sizes. Pécs is the first prototype area; other areas are selected later. Measure both algorithms on identical graph extracts and comparable time budgets. For stochastic search, repeat runs with recorded seeds; summarize mean/median, variability and failures. Do not assert that the genetic algorithm is superior until results demonstrate it.

## 7. Open questions, risks and delivery plan

| Question / risk                                                | Impact                                         | Proposed resolution                                                                                                              |
| -------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| OSM surface and access tags may be incomplete or inconsistent. | Wrong route suitability.                       | Document tag-handling rules; expose unknown categories; test edge cases.                                                         |
| Elevation is not guaranteed in the base graph.                 | Misleading ascent metrics.                     | Choose a licensed elevation source, record resolution and coverage; omit ascent where unavailable.                               |
| Loop candidates may double back or repeat many edges.          | Poor route quality.                            | Add repeated-edge penalty and directionally distributed waypoints; compare diversification strategies.                           |
| Some requests may have no feasible loop.                       | User disappointment.                           | Bound searches and report no-route outcome with suggestions for relaxing preferences.                                            |
| Genetic optimization may be slow or inconsistent.              | Limited usability or weak research comparison. | Benchmark early, use repeatable seeds and defined compute budgets; consider simulated annealing if needed.                       |
| Large OSM graphs are costly to process.                        | High memory and latency.                       | Restrict regions initially; cache graphs and shortest paths; consider more scalable routing backends if benchmarks justify them. |
| Users may interpret scores as objective truth.                 | Misunderstood rankings.                        | Display raw route metrics and preference weights; document trade-offs.                                                           |
| Stack and schedule are not mandated by the assignment.         | Overengineering before prototype.              | Treat them as proposals, adjust after feasibility and supervisor review.                                                         |

**Indicative 12-month delivery:** M1–2 OSM graph and data pipeline; M3–4 valid Pécs loop baseline and approximately 5 km demonstration; M5–7 multi-criteria scoring, genetic comparison and evaluation; M8–9 FastAPI and browser map; M10–12 persistence if useful, scale/reliability testing, documentation and final presentation. Each stage should preserve a runnable end-to-end prototype.
