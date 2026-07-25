# 30_GEOMETRY_ALGORITHMS

1. Introduction

What this concept is

Computational geometry covers algorithms for points, segments, polygons: convex hull, line intersection, nearest neighbors, sweep-line, Voronoi diagrams.

Why it exists

Spatial data and geometric queries are foundational in graphics, GIS, robotics, and CAD.

Key algorithms

- Convex hull (Graham scan, monotone chain) O(n log n)
- Line sweep for segment intersection
- KD-trees for nearest neighbor queries


2. Example: convex hull (Monotone chain)

Brief code sketch and explanation; compute lower and upper hull with sorting.


3. Interview Qs

- Implement convex hull, check polygon intersection, point in polygon (winding or ray-casting)


4. 5-min revision

Sort points then build hull; use robust predicates to avoid precision issues.