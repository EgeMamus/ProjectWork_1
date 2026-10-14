---
name: "Ege MAMUS"
neptun: "S7FBKT"
id: "2026-OG-04"
supervisor: "Gábor OLÁH"
github: "https://github.com/EgeMamus/ProjectWork_1.git"
---

# Optimized Planning of Personalized Running and Cycling Loops Based on OpenStreetMap Data

Runners and cyclists often want a route of a specific length that starts and ends at the same location, while also respecting preferences for elevation, road surface and terrain. Conventional point-to-point shortest-path routing does not solve this closed-loop, multi-criteria problem directly. This project will develop a web-based route-planning application that generates and compares personalized running and cycling loops from OpenStreetMap (OSM) data. Users will choose a starting point, target distance, activity type and preferences, then receive one or more loop routes visualized on a map with distance, cumulative elevation gain and surface summaries. The project includes an experimental comparison of at least two route-generation approaches.

## Objectives

- **Primary objective:** Build a functional, version-controlled application for personalized closed-loop running and cycling route planning.
- **Target users / stakeholders:** Recreational runners and cyclists seeking custom routes; students and researchers interested in multi-criteria graph optimization; project supervisor/evaluators.
- **Measurable success criteria:**
  - Given a valid start point and target distance, generate at least one traversable loop returning to the starting graph node, or clearly report that none was found under the current constraints.
  - Show routes on an interactive map, with total length, elevation gain (when elevation data is available) and surface-type distribution, including an explicit unknown category when data is missing.
  - Support hard constraints or configurable weights for distance, elevation gain, surface and terrain suitability; distinguish running from cycling access restrictions.
  - Implement and compare at least two route-generation/optimization approaches using route quality, execution time, valid-loop rate and scalability on reproducible test cases.
  - Include automated tests for graph processing, route closure, route statistics, cost calculation and key algorithm behavior.
- **Constraints:** Use OSM-based road/path networks; keep data processing and optimization separate from visualization; maintain Git-based version control; do not commit secrets, personal or production data.

## Proposed technical and algorithmic approach

- **Graph model:** Directed, attributed road/path graph: vertices represent junctions or segment endpoints; edges record length, allowed activity modes, available surface/terrain tags and optional elevation-derived metrics. Preserve direction and access restrictions.
- **Initial routing baseline:** Generate geographically distributed candidate waypoints, join them with shortest-path searches (A* or Dijkstra), return to the start, then reject or rank loops by feasibility and multi-criteria cost. A* and Dijkstra are path-finding components, not complete loop-generators on their own.
- **Second approach (planned):** A genetic algorithm that evolves waypoint sequences or other valid route representations; reconstructs each candidate via legal graph paths; and scores it with the same evaluation rules as the baseline. Its benefit must be measured rather than assumed. Simulated annealing is a possible later alternative.
- **Evaluation function (initial design):** Weighted, normalized penalties for target-distance deviation, undesired uphill gain, unsuitable surfaces/terrain and repeated segments, subject to hard constraints on graph access, closed-loop validity and any user-selected prohibitions. Weight settings and distance tolerance will be tested and refined.
- **Preferred implementation stack (provisional):** Python + OSMnx for OSM graph preparation; NetworkX for graph paths and experiments; FastAPI for a REST API; React + TypeScript and MapLibre GL JS for the interactive map; pytest for tests; Git/GitHub for versioning. PostgreSQL/PostGIS may be introduced later for persistent spatial data; it is not required for the first prototype. Elevation needs an additional appropriately licensed data source where OSM alone is insufficient.

## Scope

### In scope

- Import and preprocess OSM paths and roads for an initial test area (Pécs), then additional urban and mixed-terrain regions.
- Support running/cycling modes, start point, target distance and distance tolerance, and elevation/surface/terrain preferences.
- Produce closed-loop candidate routes, score and rank them, and show route geometry and descriptive statistics.
- Compare waypoint-based A\*/Dijkstra routing against a genetic waypoint optimization method (subject to prototype feasibility), with common workloads and metrics.
- Modular routing engine, API and browser-based map interface; repeatable tests and experiments.

### Out of scope for the first version

- Turn-by-turn live navigation, accounts/social features, wearable integrations and native mobile apps.
- Guaranteed optimal loops or complete coverage of incomplete OSM surface/elevation information.
- Real-time weather/traffic, popularity and route-history personalization (potential extensions, not required).

## Notes

The official assignment requires at least two approaches and specifies no programming language or algorithm. The technology stack, genetic-algorithm choice, detailed cost formulation and monthly schedule above are **proposed project design decisions**, to be revisited after measured prototypes. Unknown OSM tags and absent elevation data must not be silently interpreted as favorable conditions. Name/Neptun and repository fields remain to be completed by the student.
