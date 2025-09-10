---
layout: post
title: "Modeling Heat Sources for Heat Conduction in Unreal Engine"
date: 2025-09-09
---

Good day,

One of the most important parts of creating a real-time 3D Heat Conduction Simulation is modeling the heat source as accurately as possible. If the system is in thermal equillibrium at time t,
then for all future time the only way for heat to conduct is there to be heat input somewhere.

Now, the type of heat input model that we choose depends largely on the application area we are working with. For example, if our goal is to simulate welding 
- specifically shielded metal arc welding(SMAW), then there are many well established scientific models to work with for that purpose.

For this project, I would like to retain flexibility in the structure used to hold the heat source logic so that various applications including welding could be simulated.

I'll list a few heat conduction applications of interest:
- welding
- Blacksmithing
- Metal melting (think furnaces and foundries)
- Game applications (something like a heat ray, etc.)
- Heat Transfer Analysis (If some kind of heat is applied, how does it affect the work object)
- Heat Transfer processes during Nuclear Fission and Nuclear fusion (note, the heat transfer mechanisms involved in these nuclear processes are mostly convection and radiation,
although anything the burning hot materials come in physical contact with would be subject to the laws of heat conduction)
- Cooking (this and some other areas is less of interest for me, but it would be possible to look at this)
- Analysing the effectiveness of insulations such as for houses
- improving temperature regulation of houses (relies on heat conducting sensors)
- heat sink design and Analysis
- electrical component design and analysis (with respect to heat effects)
- heat from heat engines can be analyzed and heat profiles can be used for analysis

It occurs to me, that I would like to explore not only simulating heat conduction, but also convection by way of computational fluid dynamics(CFD) and several other processes.

There are plenty of computer aided enginering and design systems on the market, but it would be interesting to see a real-time tool like this transform an Unreal Engine(UE) Plugin or Project into
an analysis or teaching tool, or even as a way for others trying to create unique or specific real life experiences perhaps within games.

But I digress, as we move forward in this post, I will pick one of a few application areas to simulate initially, and from there design and Implement heat input models that make sense for the
current scope of the project.

I will look at and probably go through details of various research papers in what will probably be a multi part post series on heat sources and heat input models.

As for the implementation, it is my belief that the component system in UE allows for a comfortable amount of encapsulation and modularity. Also, in the event that I want to use or expose pieces of a
system to blueprints, it would allow for a smooth transition. Since performance is a critical issue in this kind of simulation, I will default to not using blueprints and doing so selectively as desired.

I go into more detail about my thoughts on and how to implement Components in C++ in UE <a href="{% post_url 2025-09-09-components-in-ue-c++ %}">here</a>
if you are interested.

$$
\begin{aligned}
&\textbf{function } \text{GetAdjacentFaces}(\text{faceRegistry},\; \text{key}): \\
&\quad \textbf{if } \text{key exists in faceRegistry}: \quad \textbf{return } \text{all entries mapped to key} \\
&\quad \textbf{else}: \quad \textbf{return } \emptyset
\end{aligned}
$$



See you in the next post,
<br>Cary