---
layout: post
title: "Part2 - Enforcing Continuity and AMR in Unreal Engine"
date: 2025-09-01
---

Good day,

Last time, I mentioned that I will go into the remaining steps on enforcing continuity in the context of Adaptive Mesh Refinement for use with mesh based field solving methods such as FEM. Let's jump right into it.

## Step 2 - Get Adjacent Faces

Step 2 will be to find adjacent faces to any face of any element. So, the map of face hashes to keys should be iterated through, and for any face that has more than one entry(values of the map), each entry that matches the input face becomes a candidate for continuity enforcement.

$$
\begin{aligned}
&\textbf{function } \textbf{GetAdjacentFaces}(\text{FaceMap}, k): \\
&\quad \textbf{if } k \in \text{FaceMap}: \textbf{return } \text{FaceMap}[k] \\
&\quad \textbf{else}: \\
&\qquad \textbf{return } \emptyset
\end{aligned}
$$


## Step 3 - Classify two faces as Coarse or Refined

For step 3, after getting two adjacent faces, the next step is to determine if each face is either at a coarse level or refined level so we can meet the condition for needing enforcement. Here is how that process might look:

$$
\begin{aligned}
&\textbf{function } \text{ClassifyCoarseRefined}(a, b): \\
&\quad \ell_a \gets \text{Level}(\text{Elements}[a.\text{ElemIndex}]) \\
&\quad \ell_b \gets \text{Level}(\text{Elements}[b.\text{ElemIndex}]) \\
&\quad \textbf{if } \ell_a = \ell_b: \textbf{return } (\text{SameLevel}, a, b) \\
&\quad \textbf{if } \ell_a < \ell_b: \textbf{return } (\text{CoarseRefined}, a, b) \\
&\quad \textbf{else: } \textbf{return } (\text{CoarseRefined}, b, a)
\end{aligned}
$$

## Step 4 - Register the Interfaces

If two of the faces are classified as coarse-refined, then register that interface. One entry suggests a boundary, greater than two suggests handling of non-manifold is required.

RegisterInterace flags the two faces for continuity enforcement.

$$
\begin{aligned}
&\textbf{FindAndRegisterInterfaces} \\
&\quad \textbf{for each } k \in \text{FaceMap:} \\
&\qquad \text{Adj} \gets \text{GetAdjacentFaces}(\text{FaceMap}, k) \\
&\qquad \textbf{if } |\text{Adj}| = 2: \\
&\qquad\quad (t, X, Y) \gets \text{ClassifyCoarseRefined}(\text{Adj}[0], \text{Adj}[1]) \\
&\qquad\quad \textbf{if } t = \text{CoarseRefined}: \\
&\qquad\qquad \text{RegisterInterface}(k, \text{Coarse}=X, \text{Refined}=Y) \\
&\qquad \textbf{else if } |\text{Adj}| = 1: \\
&\qquad\quad \text{RegisterBoundary}(k, \text{Adj}[0]) \\
&\qquad \textbf{else}: \\
&\qquad\quad \text{HandleNonManifold}(k, \text{Adj}) \\
\\
&\textbf{procedure } \text{RegisterInterface}(k, \text{Coarse}, \text{Refined}): \\
&\quad \text{InterfaceRegistry.push}(\langle k, \text{Coarse}, \text{Refined}\rangle) \\
\\
&\textbf{procedure } \text{RegisterBoundary}(k, A): \quad \text{BoundaryRegistry.push}(\langle k, A\rangle) \\
&\textbf{procedure } \text{HandleNonManifold}(k, \text{Adj}): \quad \text{NonManifoldLog.push}(\langle k, \text{Adj}\rangle)
\end{aligned}
$$


## Step 5 - Continuity Enforcement

First, enforce continuity along the field using the constraint method. Then, enforce continuity on the mesh via crack patching. That is, subdivide the coarse face or emit stitch patches so the vertices match the refined face.

$$
\begin{aligned}
&\textbf{CalculateSolverConstraints} \\
&\quad \textbf{for each } I \in \text{InterfaceRegistry}: \\
&\qquad (k, \text{Coarse}, \text{Refined}) \gets I \\
&\qquad M \gets \text{MidpointsOnFace}(k) \\
&\qquad C \gets \text{FaceCenter}(k) \\
&\qquad \textbf{for each } m \in M:\;\; \text{AddConstraint}\!\left(u_m = \tfrac{1}{2}(u_{v_1}+u_{v_2})\right) \\
&\qquad \textbf{if } C \text{ exists}:\;\; \text{AddConstraint}\!\left(u_C = \tfrac{1}{4}(u_{v_1}+u_{v_2}+u_{v_3}+u_{v_4})\right) \\
&\qquad \text{// later applied via static condensation or multipliers} \\
\\
&\textbf{BuildRenderPatches} \\
&\quad \textbf{for each } I \in \text{InterfaceRegistry}: \\
&\qquad (k, \text{Coarse}, \text{Refined}) \gets I \\
&\qquad M \gets \text{MidpointsOnFace}(k), \;\; C \gets \text{FaceCenter}(k) \\
&\qquad \text{EmitPatchGeometry}(k, M, C) \quad \text{// subdivide coarse face in the render mesh only}
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{procedure } \text{EmitPatchGeometry}(k,\;\text{coarse},\;\text{faceIdx},\;\text{refinedParent}) \\
&\quad n_u \gets 2,\;\; n_v \gets 2 \quad \text{// uniform 2×2 refinement on neighbor face} \\
\\
&\quad \text{// 1) Build a 3×3 parametric grid on the coarse face} \\
&\quad \text{Vgrid} \gets \emptyset \\
&\quad \textbf{for } i=0..n_u: \\
&\qquad u \gets i / n_u \\
&\qquad \textbf{for } j=0..n_v: \\
&\qquad\quad v \gets j / n_v \\
&\qquad\quad (p,n,t,uv) \gets \text{SampleCoarseFace}(\text{coarse},\;\text{faceIdx},\;u,\;v) \\
&\qquad\quad \text{Vgrid}[i,j] \gets (p,n,t,uv) \\
\\
&\quad \text{// 2) Snap shared border to refined neighbor} \\
&\quad \text{sharedEdge} \gets \text{WhichBorderOfFace}(\text{faceIdx},\;\text{coarse},\;\text{refinedParent}) \\
&\quad E \gets \text{RefinedEdgeVerticesOnInterface}(\text{refinedParent},\;\text{faceIdx}) \\
&\quad \textbf{for } q=0..2: \\
&\qquad (i^*,j^*) \gets \text{MapEdgeSlotToGridIndex}(\text{sharedEdge},\;q,\;n_u,\;n_v) \\
&\qquad \text{Vgrid}[i^*,j^*].p \gets E[q].p \\
&\qquad \text{Vgrid}[i^*,j^*].n \gets \text{BlendNormals}(\text{Vgrid}[i^*,j^*].n,\;E[q].n) \\
&\qquad \text{Vgrid}[i^*,j^*].uv \gets E[q].uv \\
\\
&\quad \text{// 3) Triangulate the 2×2 grid cells into 8 triangles} \\
&\quad V \gets \text{FlattenGrid}(\text{Vgrid}),\;\; I \gets \emptyset \\
&\quad \textbf{for } i=0..(n_u-1): \\
&\qquad \textbf{for } j=0..(n_v-1): \\
&\qquad\quad a \gets \text{Idx}(i,   j) \\
&\qquad\quad b \gets \text{Idx}(i+1,j) \\
&\qquad\quad c \gets \text{Idx}(i+1,j+1) \\
&\qquad\quad d \gets \text{Idx}(i,   j+1) \\
&\qquad\quad I.\text{push}([a,b,c]),\; I.\text{push}([a,c,d]) \\
\\
&\quad \textbf{return } (V,I) \quad \text{// render-only patch; solver mesh unchanged} \\
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{function } \text{SampleCoarseFace}(\text{coarse},\;\text{faceIdx},\;u,v): \\
&\quad \text{return bilinear or isoparametric sample of position, normal, tangent, uv} \\
\\
&\textbf{function } \text{WhichBorderOfFace}(\text{faceIdx},\;\text{coarse},\;\text{refinedParent}) \to \{\text{LEFT,RIGHT,BOTTOM,TOP}\} \\
\\
&\textbf{function } \text{RefinedEdgeVerticesOnInterface}(\text{refinedParent},\;\text{faceIdx}): \\
&\quad \text{return ordered list of 3 edge vertices (0, 0.5, 1)} \\
\\
&\textbf{function } \text{MapEdgeSlotToGridIndex}(\text{sharedEdge}, q,\; n_u,\; n_v): \\
&\quad \text{// map edge slot q to grid index (i^*,j^*) for the 3×3 patch} \\
\\
&\textbf{function } \text{BlendNormals}(n_a,n_b): \quad \textbf{return } \frac{n_a+n_b}{\|n_a+n_b\|} \\
\end{aligned}
$$


Once step 5 is completed, there should be continuity in the field for the solver as well as continuity in the mesh for rendering. That sastisfies the goal of this short post series. At some point I may discuss it further, but that is all for now.

Next time, I may add some more visual content to this subject or work on designing and developing other pieces to the AMR system. There is still quite a list of things that need to be done - as I decide when and what order, I will clarify it within the posts.

See you in the next post,
<br>Cary