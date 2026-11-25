---
project_outline: "[[00-project/01-project-outline]]"
project_id: "2026-OG-04"
---

<!-- 5xl -->

# Personalized Running & Cycling Loops

Optimized Planning Based on OpenStreetMap Data  
Semester 1 · Presentation 2

Author: Ege MAMUS  
Neptun: S7FBKT  
Supervisor: Gábor OLÁH  
Project ID: 2026-OG-04

---

## Specification summary

- Plan running and cycling routes that return to the starting point.
- Match the requested distance, ascent, surface and terrain preferences.
- Show ranked loop options and route statistics on a web map.
- Start in **Pécs** and compare two routing methods on the same requests.

**Semester 1 status:** Requirements and initial design are documented. The routing engine, web app and experiments are planned for semester 2.

---

## Main user journeys and requirements

**Runner or cyclist**

1. Choose an activity, pick a start and set a distance and preferences.
2. Request a loop. For example, choose a **5 km run in Pécs** with paved paths and little climbing.
3. Compare distance, ascent and surface breakdown on the map.

**Evaluator**

Run both methods on the same graph and requests. Compare route quality, runtime, valid-loop rate and scalability.

---

## Required route behavior

- Validate the input and explain any start-point adjustment to the network.
- Check that every segment permits the chosen activity and direction, and that the route returns to the same network point.
- Apply hard limits before ranking by distance, ascent, surface, terrain and repeated segments.
- Mark missing surface or elevation data. If the search finds no valid loop, show a clear result and suggest which preferences could be relaxed.

---

## Initial technical proposal

**Baseline · Waypoints + A\*/Dijkstra**  
Sample waypoints in different directions, then join them with permitted paths and return to the start.

**Alternative · Genetic waypoint optimization**  
Evolve waypoint sequences and build routes with the same shortest-path routine.

**Shared evaluator**  
Validate each loop, then score it using the same criteria for both methods.

---

## Proposed system architecture

- **Data:** OSMnx prepares a directed OpenStreetMap graph. A separate elevation source supplies height data where available.
- **Routing:** Python and NetworkX build candidate loops, check constraints and calculate metrics.
- **Delivery:** FastAPI returns route geometry and statistics. React, TypeScript and MapLibre display them on the map.

---

## Semester-2 implementation plan

1. **Build the baseline.** Prepare the Pécs graph and demonstrate a valid loop of about 5 km. Test closure and activity access with pytest.
2. **Compare the methods.** Add shared scoring and genetic search. Benchmark on the same graph and requests with fixed budgets and recorded seeds.
3. **Build the web demo.** Connect the routing API to the interactive map, then test larger areas and document the results.

---

## Open risks and next steps

- **Incomplete OSM tags or elevation:** Define how to handle missing data and select a licensed elevation source.
- **Repetitive or infeasible routes:** Penalize repeated segments, limit search time and record no-route outcomes.
- **Slow or variable genetic search:** Benchmark early and consider simulated annealing if needed.

**Next step:** Review the access rules and test requests with the supervisor, then build the first Pécs baseline.
