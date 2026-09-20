\---

name: Ege MAMUŞ
neptun: S7FBKT
id: 2026-OG-04
---

# Optimized Planning of Personalized Running and Cycling Loops Based on OpenStreetMap Data

## My interpretation of the brief

The project is a map-based route-planning application that generates personalised running or cycling loops from a selected starting point. Unlike a standard point-to-point route planner, every route must return to its starting point while remaining close to the requested distance and respecting preferences such as elevation gain, road surface, and terrain type.

The main challenge is representing OpenStreetMap data as a graph and optimising several potentially conflicting criteria at the same time. The route-generation and optimisation logic must be separated from the map interface, and the resulting routes must be presented with clear statistics about distance, elevation, and surface distribution.

## Why I am a good fit for this project

I believe this project would be a strong addition to my CV because it demonstrates algorithm development, graph-based problem solving, geospatial data processing, optimisation, testing, and web visualisation. It would also contribute significantly to my education by allowing me to apply programming and mathematical concepts to a useful real-world problem. Developing a complete route-planning system would help me improve both my technical knowledge and my ability to design and evaluate a larger software project.

## Relevant experience and background

I am a Computer Engineering student with experience in Python, HTML, and CSS. My studies have given me a foundation in programming, problem solving, and working with structured data. Python provides a suitable starting point for graph algorithms and OpenStreetMap processing, while my web-development knowledge can support the map-based interface. I am also familiar with Git and would use version control throughout the project.

## Proposed approach

I would first obtain a limited pilot area's road network from OpenStreetMap and convert it into a graph containing distance, surface, terrain, accessibility, and elevation-related attributes where available. I would define a multi-criteria cost function whose weights are derived from the user's selected activity and preferences.

I would compare at least two loop-generation approaches, such as waypoint-based candidate generation with shortest-path connections and a heuristic or stochastic search that improves complete loop candidates. The comparison would use route quality, distance error, preference satisfaction, execution time, and scalability. The optimisation component would be implemented separately from a web interface that accepts user preferences and displays one or more routes with their statistics on an interactive map. Automated tests would cover graph filtering, cost calculation, loop closure, and route-statistic calculations.

## Initial plan

1. Define the target users, route preferences, MVP boundaries, and measurable route-quality criteria.
2. Research OpenStreetMap data, graph construction, elevation sources, and candidate route-generation methods.
3. Build the graph-processing pipeline and multi-criteria cost model for a limited pilot area.
4. Implement and compare two route-generation or optimisation approaches.
5. Develop the map-based interface and display alternative routes with distance, elevation, and surface statistics.
6. Test performance and route quality, document limitations, and evaluate the application with representative scenarios.

## Additional information

I would maintain the code and documentation in a Git repository and keep external-service keys and personal data outside the repository. I would first deliver a reliable MVP for a limited geographic area before considering extensions such as weather, traffic, popularity data, or previous user routes.

