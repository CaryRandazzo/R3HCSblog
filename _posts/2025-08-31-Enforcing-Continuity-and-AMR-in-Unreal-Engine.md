---
layout: post
title: "Enforcing Continuity and AMR in Unreal Engine"
date: 2025-08-31
---

Good day,

Recently, I had been thinking about the construction of Adaptive Meshing Systems for R3HCS and wanted to describe and record my learning, understanding, and design for it.

The more I work with the Unreal Engine, the more I am learning nuances and advantages to one approach over the other. Today, I have been going over and researching the process by which continuity across mesh objects, specifically faces for hexahedral elements, is enforced during Adaptive Mesh Refinement (AMR) for Finite Element Method(FEM) based solutions.

The primary issue is that when a coarse element shares a face with refined elements, that face and the other face that shares that space must be geometrically consistent, or else it will cause rendering issues, solver issues, and interpolation and surface smoothing issues.

Three of the main methods for enforcing continuity between coarse and refined faces in the mesh are the methods of Hanging Node Constraints (FEM style), Conforming Refinement (refine the neighbors, I believe it was called red-green refinement), and Tesselation at runtime. Enforcing continuity in one of these ways is necessary for rendering AMR for heat conduction, deformations, or physics based shading for example, as well as avoiding visible seams, cracks, or inconsistent normals between adjacent elements of different refinement levels.

## Continuity Enforcement Options

There is a difference between solution continuity and physical mesh continuity. For the former, I will be using the common method of hanging node constraints.

- **Hanging Node Constraints** (continuity of the solution field) - Used in finite element analysis (FEA) to maintain the continuity and desired properties of the discrete solution space, especially on adaptively refined meshes where elements of different sizes meet. It enforces relationships between degrees of freedom (DOF) on nodes located along the edges or faces of larger elements that are surrounded by smaller, refined elements, ensuring the overall solution remains physically plausible despite the mesh discontinuity.

For the latter, I have encountered several options including

- **Crack Patching** - A targeted fix for discontinuities (cracks) at the boundaries between patches of different resolution. Crack patching adds stitch geometry or enforces matching tessellation factors along shared edges to make the boundary consistent. It should eliminate visible gaps in the mesh at the coarse-refined interfaces due to refinement.

- **Conforming Refinement** (red-green refinement, continuity of the mesh) - A mesh generation techniques in computational analysis that adapt a mesh to a specific geometry while maintaining continuity across element boundaries, preventing issues like hanging nodes.

- **Tesselation** (at runtime, continuity of the mesh) - Tessellation is the dividing of datasets of polygons (sometimes called vertex sets) presenting objects in a scene into suitable structures for rendering

My preference would be to keep unrefined neighboring elements coarse while striving for efficiency, so I will choose to use the first of those mentioned methods.

## Specific Example

Let’s think about a specific example. We have a mesh and somewhere in the mesh AMR was triggered based on field error and a piece of the mesh was refined. On the boundary of those refined elements, coarse and refined elements exist whose nodes do not necessary match.

First, the coarse element’s face has 4 nodes, but now the matching coarse neighbor with 4 nodes has been refined to 4 elements (on the boundary) where there are now 4 vertices per element on that shared face boundary. All 4 refined elements will create a set of 9 unique vertices, 4 of which is shared between it and the coarse face, thus, the coarse face needs vertices added in the same way the refined elements were created (using midpoints).

Topological refinement where we split the coarse block face to match the refined face should be best for physics simulation continuity.

<p align="center">
    <img src="https://caryrandazzo.github.io/assets/img/test.png">
  <br/>
  <em>Figure: 2D Example of a Hanging Node. Coarse face (4 nodes) adjacent to refined face (2×2 → 9 nodes) with shared edge.</em>
</p>

## Design: Mapping Shared Faces

### Step 1 Implementation

So, the first step is to construct a method to identify and reference faces on elements for detecting if a face is shared between elements. I have been using hexahedral elements for simplicity.


$$
\begin{aligned}
&\text{FaceMap} : \text{HashTable from FaceKey} \to \text{List of } (\text{ElemID}, \text{FaceIdx}) \\
&\text{FaceKey} := \text{canonical 4-tuple of node IDs for a face (orientation-invariant)} \\
\\
&\textbf{function } \text{ComputeFaceKey}(e, f): \\
&\quad V \gets \text{FaceNodes}(e, f) \quad \text{(4 node IDs)} \\
&\quad V_{\text{sorted}} \gets \text{SortAscending}(V) \\
&\quad \textbf{return } \langle V_{\text{sorted}}[0], V_{\text{sorted}}[1], V_{\text{sorted}}[2], V_{\text{sorted}}[3] \rangle \\
\\
&\textbf{function } \text{FaceNodes}(e, f): \\
&\quad F \gets \{ \\
&\qquad 0:\langle 0,1,2,3\rangle,\; 1:\langle 4,5,6,7\rangle,\; 2:\langle 0,1,5,4\rangle, \\
&\qquad 3:\langle 2,3,7,6\rangle,\; 4:\langle 0,3,7,4\rangle,\; 5:\langle 1,2,6,5\rangle \\
&\quad \} \\
&\quad \textbf{return } \langle e.\text{Nodes}[F[f][0]], e.\text{Nodes}[F[f][1]], e.\text{Nodes}[F[f][2]], e.\text{Nodes}[F[f][3]] \rangle \\
\\
&\textbf{for each element } e \in \text{Elements:} \\
&\quad \textbf{for each face } f \in e: \\
&\qquad k \gets \text{ComputeFaceKey}(e, f) \\
&\qquad \text{FaceMap}[k] \gets \text{FaceMap}[k] \cup \{(e.\text{id}, f)\} \\
\\
&\textbf{for each } k \in \text{FaceMap:} \\
&\quad \textbf{if } |\text{FaceMap}[k]| = 2: \\
&\qquad \text{Identify coarse vs refined element} \\
&\qquad \text{Register interface for constraint enforcement}
\end{aligned}
$$

This blog outlines the general algorithm. Implementation details are proprietary and remain part of the closed-source R3HCS project at this time.

That's all for now. In the next blog post, I will go into the remaining steps on enforcing continuity in the context of Adaptive Mesh Refinement for use with mesh based field solving methods such as FEM.

See you in the next post,
<br>Cary