$$
\begin{aligned}
&\textbf{function } \text{GetAdjacentFaces}(\text{FaceMap}, k): \\
&\quad \textbf{return } \text{FaceMap}[k] \quad \text{// list of } (\text{ElemID}, \text{FaceIdx})
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{function } \text{ClassifyCoarseRefined}(a, b): \\
&\quad \ell_a \gets \text{Level}(\text{Elements}[a.\text{ElemID}]) \\
&\quad \ell_b \gets \text{Level}(\text{Elements}[b.\text{ElemID}]) \\
&\quad \textbf{if } \ell_a = \ell_b: \textbf{return } (\text{SameLevel}, a, b) \\
&\quad \textbf{if } \ell_a < \ell_b: \textbf{return } (\text{CoarseRefined}, a, b) \\
&\quad \textbf{else: } \textbf{return } (\text{CoarseRefined}, b, a)
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{for each } k \in \text{FaceMap}: \\
&\quad \text{Adj} \gets \text{GetAdjacentFaces}(\text{FaceMap}, k) \\
&\quad \textbf{if } |\text{Adj}| = 2: \\
&\qquad (t, X, Y) \gets \text{ClassifyCoarseRefined}(\text{Adj}[0], \text{Adj}[1]) \\
&\qquad \textbf{if } t = \text{CoarseRefined}: \\
&\qquad\quad \text{RegisterInterface}(k, \text{Coarse}=X, \text{Refined}=Y) \\
&\quad \textbf{else if } |\text{Adj}| = 1: \\
&\qquad \text{RegisterBoundary}(k, \text{Adj}[0]) \\
&\quad \textbf{else}: \\
&\qquad \text{HandleNonManifold}(k, \text{Adj})
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{procedure } \text{RegisterInterface}(k, \text{Coarse}, \text{Refined}): \\
&\quad \text{// Derive interface points implied by refinement (edge midpoints, face center)} \\
&\quad M \gets \text{MidpointsOnFace}(k) \\
&\quad C \gets \text{FaceCenter}(k) \\
&\quad \text{// Build constraint stencils (examples below are Q1-like)} \\
&\quad \textbf{for each } m \in M: \\
&\qquad \text{AddConstraint}\big(u_m = \tfrac{1}{2}(u_{v_1} + u_{v_2})\big) \\
&\quad \textbf{if } C \text{ exists}: \\
&\qquad \text{AddConstraint}\big(u_C = \tfrac{1}{4}(u_{v_1}+u_{v_2}+u_{v_3}+u_{v_4})\big) \\
&\quad \text{// Defer solver coupling choice } (\text{static condensation vs. multipliers})
\end{aligned}
$$

$$
\begin{aligned}
&\textbf{function } \text{MidpointsOnFace}(k): \textbf{return } \{ \text{mid}(v_i, v_j) \; \text{for each face edge} \} \\
&\textbf{function } \text{FaceCenter}(k): \textbf{return } \text{centroid}(v_1,v_2,v_3,v_4) \\
&\textbf{procedure } \text{AddConstraint}(\cdot): \text{enqueue in constraint system}
\end{aligned}
$$