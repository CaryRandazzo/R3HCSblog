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
&\textbf{procedure } \text{FindAndRegisterInterfaces}: \\
&\quad \textbf{for each } k \in \text{FaceMap}: \\
&\qquad \text{Adj} \gets \text{GetAdjacentFaces}(\text{FaceMap},\; k) \\
&\qquad \textbf{if } |\text{Adj}| = 2: \\
&\qquad\quad (t,\; A,\; B) \gets \text{ClassifyCoarseRefined}(\text{Adj}[0],\; \text{Adj}[1]) \\
&\qquad\quad \textbf{if } t = \text{CoarseRefined} \textbf{ and } A.\text{isOnSurface} \land B.\text{isOnSurface}: \\
&\qquad\qquad \text{RegisterInterface}(k,\; \text{Coarse}=A,\; \text{Refined}=B) \\
&\qquad \textbf{else if } |\text{Adj}| = 1: \\
&\qquad\quad \text{RegisterBoundary}(k,\; \text{Adj}[0]) \\
&\qquad \textbf{else}: \\
&\qquad\quad \text{HandleNonManifold}(k,\; \text{Adj}) \\
\\
&\textbf{procedure } \text{RegisterInterface}(k,\; \text{Coarse},\; \text{Refined}): \\
&\quad \text{InterfaceRegistry.push}(\langle k,\; \text{Coarse},\; \text{Refined} \rangle) \\
\\
&\textbf{procedure } \text{RegisterBoundary}(k,\; A): \quad \text{BoundaryRegistry.push}(\langle k,\; A \rangle) \\
&\textbf{procedure } \text{HandleNonManifold}(k,\; \text{Adj}): \quad \text{NonManifoldLog.push}(\langle k,\; \text{Adj} \rangle)
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
&\textbf{procedure } \text{EmitPatchGeometry}(k): \\
&\quad (\text{coarseBlock},\; \text{faceIdx}) \gets \text{FaceMap}[k] \\
&\quad V \gets \text{GetCornerVertices}(\text{coarseBlock},\; \text{faceIdx}) \\
&\quad M \gets \text{ComputeEdgeMidpoints}(V) \\
&\quad C \gets \text{ComputeFaceCenter}(V) \\
&\quad \textbf{return } \text{PatchGrid}(V,\; M,\; C)
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{function } \text{GetCornerVertices}(\text{block},\; \text{faceIdx}): \\
&\quad \text{return the 4 vertices defining the face} \\
\\
&\textbf{function } \text{ComputeEdgeMidpoints}(V): \\
&\quad \text{return midpoints of edges between adjacent corners in } V \\
\\
&\textbf{function } \text{ComputeFaceCenter}(V): \\
&\quad \text{return average of all 4 corner vertices in } V \\
\\
&\textbf{function } \text{PatchGrid}(V,\; M,\; C): \\
&\quad \text{return ordered list of 9 vertices (3×3 grid)} \\
&\quad \text{Layout: corners, mids, center}
\end{aligned}
$$




Once step 5 is completed, there should be continuity in the field for the solver as well as continuity in the mesh for rendering. That sastisfies the goal of this short post series. At some point I may discuss it further, but that is all for now.

Next time, I may add some more visual content to this subject or work on designing and developing other pieces to the AMR system. There is still quite a list of things that need to be done - as I decide when and what order, I will clarify it within the posts.

See you in the next post,
<br>Cary