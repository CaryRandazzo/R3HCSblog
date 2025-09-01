---
layout: post
title: "Part2 - Enforcing Continuity and AMR in Unreal Engine"
date: 2025-09-01
---

Good day,

Last time, I mentioned that I will go into the remaining steps on enforcing continuity in the context of Adaptive Mesh Refinement for use with mesh based field solving methods such as FEM. Let's jump right into it.

Step 2 will be to find adjacent faces to any face of any element. So, the map of face hashes to keys should be iterated through, and for any face that has more than one entry(values of the map), each entry that matches the input face becomes a candidate for continuity enforcement.

$$
\begin{aligned}
&\textbf{function } \text{GetAdjacentFaces}(\text{FaceMap}, k): \\
&\quad \textbf{return } \text{FaceMap}[k] \quad \text{// list of } (\text{ElemID}, \text{FaceIdx})
\end{aligned}
$$

See you in the next post,
<br>Cary