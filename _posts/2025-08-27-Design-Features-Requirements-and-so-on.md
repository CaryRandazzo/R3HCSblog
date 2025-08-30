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

<b> Hardware Used</b>
Speaking of local hardware, the hardware setup that I am currently running on and will be testing this system on is as follows:
8GB VRAM NVidia GeForce RTX 4060 TI
32GB RAM
AMD Ryzen 7 8700F 8-Core Processor 4.10GHz
Windows 11(Not using UE in Linux)

Recently, I have developed an interest i VR environments, so as necessary I may interface R3HCS with VR.

<b>Initial Design Sketch</b>
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


See you in the next post,
<br>Cary