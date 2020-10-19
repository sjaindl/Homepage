---
title: 'Route Planner'
routable: false
---

Route Planner is a lightweight Google Maps-like algorithm for finding the shortest path in a map. It is written in Python and based on A* search. I'll provide a short explanation of how the algorithm works, its runtime and space requirements.

It uses a heuristic function f, where f = g + h. g is the distance covered so far and h is the estimated distance to the goal. The calculations are based on the Euclidean distance.

The total cost is stored in a min priority queue, so the current element with the min cost can be stored and retrieved in log(V) time, each.
When the min element is the goal, the goal check is successful. It returns the path, which is backtracked by links to predecessors. This requires just constant O(1) additional storage. In total, the algorithm requires O(V) additional space and O(E * V log V) time, where E are roads (edges) and V are intersections (vertices). A deeper analysis can be found in the comments of a_star_path_search.py.

The repository is publicly available on [here](https://github.com/sjaindl/RoutePlanner).