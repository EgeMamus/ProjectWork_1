---
project_outline: "[[01-project-outline]]"
project_id: "2026-OG-04"
---

<!-- 5xl -->

# Personalized Running & Cycling Loops

Optimized Planning Based on OpenStreetMap Data

Author: Ege MAMUS
Neptun: S7FBKT
Supervisor: Gábor OLÁH  
Project ID: 2026-OG-04

---

## Problem

- Runners and cyclists often want a route of a specific distance **ending where it started**.
- Standard point-to-point shortest paths are not enough to generate useful loops.
- Route quality also depends on **elevation gain, surface and terrain**.
- A good route must respect the chosen activity's access restrictions.

**Challenge:** generate a practical, personalized closed loop rather than just the shortest path.

---

## Proposed idea

A web application that:

- Uses **OpenStreetMap** paths and roads as an attributed graph.
- Lets users select a start, distance, running/cycling mode and preferences.
- Generates and ranks **closed-loop route candidates**.
- Presents alternatives on an interactive map with distance, ascent and surface statistics.

---

## Main goal and research component

**Deliver a working routing application and compare at least two optimization approaches.**

| Approach                               | Purpose                                                       |
| -------------------------------------- | ------------------------------------------------------------- |
| **Waypoint generation + A\*/Dijkstra** | Interpretable, reproducible baseline for valid loops          |
| **Genetic waypoint optimization**      | Search for better route candidates under the same preferences |

Compare **route quality, runtime, valid-loop rate and scalability** on shared test scenarios. The second method's advantage is a hypothesis, not an assumed result.

---

## Planned algorithm logic

1. Prepare an OSM **directed road/path graph** with length, permitted mode, surface and available terrain/elevation properties.
2. Generate waypoint sequences around the start.
3. Connect waypoint pairs using legal shortest paths.
4. Return to the starting node; reject disconnected or forbidden paths.
5. Measure candidate distance, ascent, surfaces, terrain and repeated segments.
6. Rank feasible loops and present several distinct choices where possible.

**Why a loop generator?** Dijkstra/A\* alone do not solve the fixed-distance closed-loop problem.

---

## Multi-criteria route scoring

**Initial proposed objective:** minimize a weighted combination of:

- Deviation from requested distance
- Undesired elevation gain
- Disfavored road surfaces and terrain
- Repeated road segments

**Hard constraints:** route closure, legal running/cycling access and any explicit user prohibitions.

Weights and normalization will be validated experimentally; unknown surface/elevation values will be shown as unknown, not guessed.

---

## Proposed technology stack

| Component                      | Technology                  |
| ------------------------------ | --------------------------- |
| Algorithms and data processing | **Python, OSMnx, NetworkX** |
| Backend API                    | **FastAPI**                 |
| Web interface                  | **React + TypeScript**      |
| Map visualization              | **MapLibre GL JS**          |
| Automated testing              | **pytest**                  |
| Source control                 | **Git / GitHub**            |
| Later, if justified            | **PostgreSQL + PostGIS**    |

**Separation:** OSM preparation → optimization engine → API → web map. An external elevation source may be needed.

---

## Target users

| User                   | Need                                                       |
| ---------------------- | ---------------------------------------------------------- |
| Recreational runner    | A preferred-distance loop with suitable effort and surface |
| Cyclist                | A cycling-permitted loop with appropriate terrain          |
| Evaluator / supervisor | Repeatable comparison of graph-based optimization methods  |

First supported area: **Pécs**, with later expansion to larger and different road networks.

---

## Planned scope

**In scope**

- OSM network preparation and activity-specific accessibility
- Personalized closed-loop route generation
- Map visualization and basic route statistics
- Two algorithmic approaches, automated tests and benchmarks

**Not in initial scope**

- Turn-by-turn live navigation and native mobile apps
- Social features and wearable syncing
- Weather, traffic, popularity or past-route learning

---

## User journey 1: everyday route request

1. User selects **Pécs**, a start point and **5 km running**.
2. User prefers paved paths and minimal climbing.
3. App produces feasible loop options returning to the start.
4. User compares **distance, elevation and surface information**.
5. User selects and views the preferred loop on the map.

If no feasible loop exists, the app explains that clearly.

---

## User journey 2: algorithm evaluation

1. Prepare the **same OSM graph extract** and set of route requests.
2. Run waypoint + A\*/Dijkstra baseline.
3. Run genetic waypoint optimization with recorded random seeds.
4. Verify loop closure, access rules and distance quality.
5. Compare **quality, execution time, scalability and failure rate**.

Do not discard unsuccessful runs from the experimental results.

---

## Milestones: approximately one year

| Period       | Planned outcome                                     |
| ------------ | --------------------------------------------------- |
| Months 1–2   | OSM graph/data foundation                           |
| Months 3–4   | First valid ~5 km loop in Pécs                      |
| Months 5–7   | Preference scoring, second algorithm, benchmarks    |
| Months 8–9   | FastAPI and React/MapLibre application              |
| Months 10–12 | Performance, optional persistence, tests and report |

The plan will be refined after early feasibility testing.

---

## Next steps

- Study graph representations, shortest paths and OSM data tags.
- Build a small **Pécs running network** with Python + OSMnx.
- Implement and test a valid closed loop using **waypoints and A\*/Dijkstra**.
- Define the shared evaluator and benchmark scenarios **before** tuning the genetic algorithm.
- Review design decisions and scope with the supervisor.

**First concrete milestone:** a reproducible 5 km-class loop prototype, before building the full web interface.
