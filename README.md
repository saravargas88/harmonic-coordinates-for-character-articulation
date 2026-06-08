# Harmonic Coordinates for Character Articulation

Implementation of harmonic coordinates for interactive 2D mesh deformation, based on the technique used in production character animation (Pixar's "Harmonic Coordinates for Character Articulation", Joshi et al. 2007).

## What it does

Given a 2D character mesh and a surrounding cage, the system computes **harmonic coordinates** — smooth, cage-aware weights that control how each mesh vertex moves when the cage is manipulated. Moving a cage handle smoothly deforms the character mesh while preserving local shape.

## How it works

### 1. Cage triangulation
The boundary cage is triangulated using constrained Delaunay triangulation (`triangle` library), producing a mesh that fills the interior of the cage.

### 2. Solving the Laplace equation
For each cage boundary vertex, a harmonic basis function φᵢ is computed by solving the discrete Laplace equation:

```
Δφ = 0   over the cage interior
φᵢ = 1   at boundary vertex i
φᵢ = 0   at all other boundary vertices
```

Discretized using the cotangent weight Laplacian `L` and mass matrix `M` from `libigl`:

```
A = M⁻¹ L

Aff @ φ_free = -Afc @ φ_constrained
```

This sparse linear system is solved with `scipy.sparse.linalg.spsolve`.

### 3. Harmonic coordinate computation
Each object mesh vertex is located inside a cage triangle and its harmonic coordinates are computed via barycentric interpolation of the three triangle vertices' basis functions.

### 4. Interactive deformation
At runtime, deformed mesh positions are computed as:

```
v_deformed = H @ new_cage_boundary_positions
```

where `H` is the precomputed harmonic coordinate matrix (object vertices × cage handles).

## Key files

- `harmonic_coord_articulation.ipynb` — full implementation and interactive demo
- `data/woody-hi.off` — character mesh
- `data/woody-hi.cage.npy` — cage boundary vertices

## Dependencies

```
numpy
scipy
libigl (igl)
triangle
meshplot
ipywidgets
```

## Mathematical background

Harmonic coordinates generalize barycentric coordinates to arbitrary cages. Unlike linear blend skinning, they:
- Are always non-negative inside convex and non-convex cages
- Produce smooth deformations that respect cage topology
- Are precomputed once, making runtime deformation a single matrix multiply

The cotangent Laplacian discretizes the continuous Laplace–Beltrami operator on the triangulated cage, preserving the harmonic property under mesh refinement.

## References

Joshi, P., Meyer, M., DeRose, T., Green, B., & Sanocki, T. (2007). Harmonic coordinates for character articulation. *ACM Transactions on Graphics (SIGGRAPH)*, 26(3).
