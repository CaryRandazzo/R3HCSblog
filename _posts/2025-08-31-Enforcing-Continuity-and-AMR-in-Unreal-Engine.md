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
    <img src="https://caryrandazzo.github.io/R3HCSblog/assets/img/test.png">
  <br/>
  <em>Figure: 2D Example of a Hanging Node. Coarse face (4 nodes) adjacent to refined face (2×2 → 9 nodes) with shared edge.</em>
</p>

## Design: Mapping Shared Faces

### Step 1 Implementation

So, the first step is to construct a method to identify and reference faces on elements for detecting if a face is shared between elements. I have been using hexahedral elements for simplicity.


$$
\begin{aligned}
&\text{faceRegistry} : \text{HashTable from FaceKey} \to \text{List of (ElementID, FaceID)} \\
&\text{FaceKey} := \text{canonical 4-tuple of node indices representing a face (order-invariant)} \\
\\
&\textbf{function } \text{ComputeFaceKey}(\text{element}, \text{face}): \\
&\quad V \gets \text{GetFaceNodes}(\text{element}, \text{face}) \quad \text{(list of 4 node indices)} \\
&\quad V_{\text{sorted}} \gets \text{SortAscending}(V) \\
&\quad \textbf{return } V_{\text{sorted}} \\
\\
&\textbf{function } \text{GetFaceNodes}(\text{element}, \text{faceID}): \\
&\quad \text{Use standard face definitions for a hexahedral element} \\
&\quad \textbf{return } \text{list of 4 node indices for the given face} \\
\\
&\textbf{for each } \text{element} \in \text{AllElements}: \\
&\quad \textbf{for each face } f \in \text{element}: \\
&\qquad k \gets \text{ComputeFaceKey}(\text{element}, f) \\
&\qquad \text{faceRegistry}[k] \gets \text{faceRegistry}[k] \cup \{(\text{element.ID}, f)\} \\
\\
&\textbf{for each } k \in \text{faceRegistry}: \\
&\quad \textbf{if } |\text{faceRegistry}[k]| = 2: \\
&\qquad \text{Determine refinement levels of the two elements} \\
&\qquad \text{If one is coarser, register interface for continuity enforcement}
\end{aligned}
$$

This blog outlines the general algorithm. Implementation details are proprietary and remain part of the closed-source R3HCS project at this time.

That's all for now. In the next blog post, I will go into the remaining steps on enforcing continuity in the context of Adaptive Mesh Refinement for use with mesh based field solving methods such as FEM.

See you in the next post,
<br>Cary