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
&\textbf{procedure } \text{EmitPatchGeometry}(\text{coarse},\; \text{faceIdx},\; \text{sharedEdge},\; \text{refinedParent}) \\
&\quad (v_{00}, v_{10}, v_{11}, v_{01}) \gets \text{FaceCornersOrdered}(\text{coarse}, \text{faceIdx}) \\
\\
&\quad m_{u0} \gets \text{Midpoint}(v_{00}, v_{10}) \quad \text{// bottom edge} \\
&\quad m_{u1} \gets \text{Midpoint}(v_{01}, v_{11}) \quad \text{// top edge} \\
&\quad m_{v0} \gets \text{Midpoint}(v_{00}, v_{01}) \quad \text{// left edge} \\
&\quad m_{v1} \gets \text{Midpoint}(v_{10}, v_{11}) \quad \text{// right edge} \\
&\quad c \gets \text{FaceCenter}(v_{00}, v_{10}, v_{11}, v_{01}) \\
\\
&\quad grid[0,0] = v_{00},\; grid[1,0] = m_{u0},\; grid[2,0] = v_{10} \\
&\quad grid[0,1] = m_{v0},\; grid[1,1] = c,\;\;\;\;\;\;\; grid[2,1] = m_{v1} \\
&\quad grid[0,2] = v_{01},\; grid[1,2] = m_{u1},\; grid[2,2] = v_{11} \\
\\
&\quad E \gets \text{RefinedEdgeVerticesOnSharedBorder}(\text{refinedParent}, \text{faceIdx}) \\
\\
&\quad \textbf{if } \text{sharedEdge} = \text{LEFT}: \;\; slots \gets \{(0,0),(0,1),(0,2)\} \\
&\quad \textbf{else if } \text{sharedEdge} = \text{RIGHT}: \; slots \gets \{(2,0),(2,1),(2,2)\} \\
&\quad \textbf{else if } \text{sharedEdge} = \text{BOTTOM}: \; slots \gets \{(0,0),(1,0),(2,0)\} \\
&\quad \textbf{else}: \;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\; slots \gets \{(0,2),(1,2),(2,2)\} \quad \text{// TOP} \\
\\
&\quad \textbf{for } q=0..2: \\
&\qquad (i,j) \gets slots[q] \\
&\qquad grid[i,j].pos \gets E[q].pos \\
\\
&\quad V \gets \text{FlattenGrid}(grid),\;\; I \gets \emptyset \\
&\quad \textbf{for } i=0..1: \\
&\qquad \textbf{for } j=0..1: \\
&\qquad\quad a \gets Idx(i,j),\; b \gets Idx(i+1,j),\; c \gets Idx(i+1,j+1),\; d \gets Idx(i,j+1) \\
&\qquad\quad I.\text{push}([a,b,c]),\;\; I.\text{push}([a,c,d]) \\
\\
&\quad \textbf{return } (V,I)
\end{aligned}
$$



Once step 5 is completed, there should be continuity in the field for the solver as well as continuity in the mesh for rendering. That sastisfies the goal of this short post series. At some point I may discuss it further, but that is all for now.

Next time, I may add some more visual content to this subject or work on designing and developing other pieces to the AMR system. There is still quite a list of things that need to be done - as I decide when and what order, I will clarify it within the posts.

See you in the next post,
<br>Cary