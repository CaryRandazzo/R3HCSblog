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
&\textbf{function } \text{GetAdjacentFaces}(\text{faceRegistry},\; \text{key}): \\
&\quad \textbf{if } \text{key exists in faceRegistry}: \quad \textbf{return } \text{all entries mapped to key} \\
&\quad \textbf{else}: \quad \textbf{return } \emptyset
\end{aligned}
$$

For simplicity, I will skip explicit use of this in practice.


## Step 3 - Classify two faces as Coarse or Refined

For step 3, after getting two adjacent faces, the next step is to determine if each face is either at a coarse level or refined level so we can meet the condition for needing enforcement. Here is how that process might look:

$$
\begin{aligned}
&\textbf{function } \text{ClassifyCoarseRefined}(a, b): \\
&\quad \ell_a \gets \text{refinement level of element associated with } a \\
&\quad \ell_b \gets \text{refinement level of element associated with } b \\
&\quad \textbf{if } \ell_a = \ell_b: \quad \textbf{return } (\text{SameLevel}, a, b) \\
&\quad \textbf{if } \ell_a < \ell_b: \quad \textbf{return } (\text{CoarseRefined}, a, b) \\
&\quad \textbf{else:} \quad \textbf{return } (\text{CoarseRefined}, b, a)
\end{aligned}
$$

## Step 4 - Register the Interfaces

If two of the faces are classified as coarse-refined, then register that interface. One entry suggests a boundary, greater than two suggests handling of non-manifold is required.

RegisterInterace flags the two faces for continuity enforcement. Continuity on the mesh will be enforced when it is on the surface and the adjacent face is also on a block on the surface. The visible interface would have a crack without patching, and that is the target of this last part.

$$
\begin{aligned}
&\textbf{procedure } \text{FindAndRegisterInterfaces}: \\
&\textbf{procedure } \text{FindAndRegisterInterfaces}: \\
&\quad \textbf{for each } \text{shared face between two elements:} \\
&\qquad \textbf{if } \text{both sides of the face exist:} \\
&\qquad\quad \text{determine if the face is between a coarse and a refined element} \\
&\qquad\quad \textbf{if } \text{both elements are on the visible surface:} \\
&\qquad\qquad \text{mark this face for patching later} \\
&\qquad \textbf{else if } \text{only one side exists:} \\
&\qquad\quad \text{record as a boundary} \\
&\qquad \textbf{else:} \\
&\qquad\quad \text{log as a non-manifold face} \\
\\
&\textbf{procedure } \text{EnforceMeshContinuity}: \\
&\quad \textbf{for each } \text{face marked for patching:} \\
&\qquad \text{identify the coarse element and face index} \\
&\qquad \text{generate and emit a patch to ensure mesh continuity} \\
\end{aligned}
$$


## Step 5 - Continuity Enforcement

First, enforce continuity along the field using the constraint method. Then, enforce continuity on the mesh via crack patching. That is, subdivide the coarse face or emit stitch patches so the vertices match the refined face.

$$
\begin{aligned}
&\textbf{EnforceSolverContinuity} \\
&\quad \textbf{for each } \text{interface marked for solver enforcement:} \\
&\qquad \text{gather edge midpoints on the shared face} \\
&\qquad \text{optionally compute center point of the face} \\
&\qquad \textbf{for each midpoint:} \\
&\qquad\quad \text{apply solver constraint using adjacent node values} \\
&\qquad \textbf{if } \text{center point is present:} \\
&\qquad\quad \text{apply additional constraint using center and corners} \\
\\
&\textbf{procedure } \text{EnforceMeshContinuity}: \\
&\quad \textbf{for each } \text{face marked for patching:} \\
&\qquad \text{identify the coarse element and face index} \\
&\qquad \text{generate and emit a patch to ensure mesh continuity} \\
\\
&\textbf{procedure } \text{EmitPatchGeometry}(\text{interface}): \\
&\quad \text{coarseFace} \leftarrow \text{extract coarse face from interface} \\
&\quad \textbf{if } \text{coarseFace has not been subdivided}: \\
&\quad\quad \text{compute edge midpoints of the face} \\
&\quad\quad \text{compute center point of the face} \\
&\quad \text{construct patch with:} \\
&\quad\quad \text{— 4 corner vertices} \\
&\quad\quad \text{— 4 edge midpoints} \\
&\quad\quad \text{— 1 face center} \\
&\quad \text{return or submit patch for stitching}
\end{aligned}
$$

In reality, a more complex treatment of the continuity in the degrees of freedom will be needed. It will need to be determined where and how this data will affect the global or local matrices or similar of the FEM solver. Thus, AddConstraint will be reviewed at a later time.

Once step 5 is completed, there should be continuity in the mesh for rendering and we have established an introductory treatment to continuity in the field for the solver. That sastisfies the goal of this short post series. At some point I may discuss it further, but that is all for now.

Next time, I may add some more visual content to this subject or work on designing and developing other pieces to the AMR system. There is still quite a list of things that need to be done - as I decide when and what order, I will clarify it within the posts.

See you in the next post,
<br>Cary