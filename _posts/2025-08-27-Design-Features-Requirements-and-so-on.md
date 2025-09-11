---
layout: post
title: "Design, Features, Requirements and so on."
date: 2025-08-27
---

Good day,

Today I am going to go over the initial design, requirements, and general plans for this project moving forward.

As most of my recent skills and interests are with making simlations with the Unreal Engine, so the project will be a UE Project. Additionally, packaging this project into what will effectively be a tool that will facilitate heat condiction in a variety of contexts, it would make the most sense to me to package it as a plugin.

I had learned from previous exercises that making custom components in UE has its valid uses and there likely wil be some emphasis on this, but I have reently gained an interest in encapsulating features and separate pieces of code as separate includable libraries where it makes sense - separate classes in c++ if you will. Much of the system will likely require c++ only classes, but again some UE infrastructure will be necessary when the R3HCS plugin is used in other projects.

Performance will be a key requirement of this project. For it to be Real-Time it should feel as such. Thus, some thought will be put into what and how the performance is constrained and optimized. Time Complexity Analysis of algorithms will be done prior to implementation and initially the system will utilize the CPU, but the door will be open for utilizing GPU based compute shaders courtesy of UE and my local hardware GPU.

### Hardware Used

Speaking of local hardware, the hardware setup that I am currently running on and will be testing this system on is as follows:
- 8GB VRAM NVidia GeForce RTX 4060 TI
- 32GB RAM
- AMD Ryzen 7 8700F 8-Core Processor 4.10GHz
- Windows 11(Not using UE in Linux)

Recently, I have developed an interest i VR environments, so as necessary I may interface R3HCS with VR.

### Initial Design Sketch
- Unreal Engine
- Libraries
- Performance (use cpu when possible, but GPU for applying system to complex objects)
- 3D
- Transient rather than Steady State
- Heat Conduction
- Fourier's Law of Conduction in 3D
- Finite Element Method as the PDE solution method
- Some or more of the Galerkin Method as needed
- Heat Source (a hand with the power of heating and melting? iron man?)
- Target Material Information
- Input Hardware Information
- Method to initialize the R3HCS in Real-Time
- A section of code to handle the mesh and mesh related information
- A section of code to handle the actual FEM and Solver logic of the PDE
- Plugin
- Adaptive Mesh Refinement at some point
- A decision still needs to be made about what type of objects this system will be designed to handle - large complex, or relatively small? etc.

### Applications

I have several areas of application I am interested in and can choose from. I'll list a few:
- Welding
- Blacksmithing
- Metal melting (think furnaces and foundries)
- Game applications (something like a heat ray, etc.)
- Heat Transfer Analysis (If some kind of heat is applied, how does it affect the work object)
- Heat Transfer processes during Nuclear Fission and Nuclear fusion (note, the heat transfer mechanisms involved in these nuclear processes are mostly convection and radiation, although anything the burning hot materials come in physical contact with would be subject to heat conduction laws)
- Cooking (this and some other areas is less of interest for me, but it would be possible to look at this)
- Analysing the effectiveness of insulations such as for houses
- improving temperature regulation of houses (relies on heat conducting sensors)
- heat sink design and Analysis
- electrical component design and analysis (with respect to heat effects)
- heat from heat engines can be analyzed and heat profiles can be used for analysis

The first area of application that I intend to focus on is oven like heating. Now, the interesting thing about some ovens are that there is heat transfer from conduction, convection, and radiation. It is possible that most of the heat transfer happens via convective heat transfer such as with a convection oven. In some cases, the system may require only simulating one kind of 
heat transfer method, but in this case we will make a simulation that uses all of them with the first goal of making perhaps a convective oven.

### How to Model the Heat Conduction System

 To model the 3 heat  transfer processes, I will focus on a Finite Element Method(FEM) based simulation system perhaps with some AI pieces or modifications where it is useful or interesting. I'll apply convective and radiative heat transfer on the boundary of an object (workpiece) where  it then conducts heat as described by the PDE: Fourier's Law of Conduction. In this case, the 3D transient formulation is used.

Our starting point then is Fourier's Law of Conduction in 3D Transient form:

$$
\rho c_p \frac{\partial T}{\partial t} - \nabla \cdot k \nabla T = q(x,y,z,t)
$$

from there, we will begin solving the FEM on this PDE using a standard bein method such as the galerkin method.

The full details of the model design is given in a separate post here.

with the model designed, focus can now turn to  initial mplementation design decisions

### initial implementation design

- Mesh
- adaptive mesh refinement
- elements
- hardware parameters
- material parameters (of workpiece object)
- method to attach and detach components to workpieces/objects
- when and how that attachment will occur
- data access and transfer design
- algorithms and complexity
- trees, insertion, and traversal
- main solver
- ...

### A thought about future design goals

It occurs to me, that I would like to explore not only simulating heat conduction, but also convection by way of computational fluid dynamics(CFD) and several other processes.

There are plenty of computer aided enginering and design systems on the market, but it would be interesting to see a real-time tool like this transform an Unreal Engine(UE) Plugin or Project into an analysis or teaching tool, or even as a way for others trying to create unique or specific real life experiences perhaps within games.

### Outro

At a later time, I will summarize and bring this initial design document to a close, although future feature by feature design docs may come.

See you in the next post,
<br>Cary

***Updated 2025-09-10***