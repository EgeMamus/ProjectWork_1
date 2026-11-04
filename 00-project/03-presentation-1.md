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

**Goal:** Build a routing application and compare two optimization approaches.

| Approach                    | Purpose                  |
| :-------------------------- | :----------------------- |
| **Waypoint + A\*/Dijkstra** | Baseline for valid loops |
| **Genetic optimization**    | Search for better loops  |

**Compare:** route quality, runtime, valid-loop rate and scalability.

_Genetic improvement is a hypothesis, not a guaranteed result._

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

## Next steps

- Study graph representations, shortest paths and OSM data tags.
- Build a small **Pécs running network** with Python + OSMnx.
- Implement and test a valid closed loop using **waypoints and A\*/Dijkstra**.
- Define the shared evaluator and benchmark scenarios **before** tuning the genetic algorithm.
- Review design decisions and scope with the supervisor.

**First concrete milestone:** a reproducible 5 km-class loop prototype, before building the full web interface.
