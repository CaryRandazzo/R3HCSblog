---
layout: post
title: "Modeling Heat Sources for Heat Conduction in Unreal Engine"
date: 2025-09-09
tags: [HeatInput]
---

Good day,

### Introduction

One of the most important parts of creating a real-time 3D Heat Conduction Simulation is modeling the heat source as accurately as possible. If the system is in thermal equillibrium at time t, then for all future time the only way for heat to conduct is there to be heat input somewhere.

Now, the type of heat input model that we choose depends largely on the application area we are working with. For example, if our goal is to simulate welding. Specifically, shielded metal arc welding(SMAW), then there are many well established scientific models to work with for that purpose.


### Heat Input Model Design

For this project, I would like to retain flexibility in the structure used to hold the heat source logic so that various applications including welding could be simulated.

Let us choose a heat input model that fits with the (application) area and makes sense for the current scope of the project.

The application area was chosen in the <a href="{{ site.baseurl }}{% post_url R3HCSblog/2025-08-27-Design-Features-Requirements-and-so-on %}">inital design document</a>.


### Design and Implementation

I will look at and probably go through details of various research papers in what will probably be a multi part post series on heat sources and heat input models.

As for the implementation, it is my belief that the component system in UE allows for a comfortable amount of encapsulation and modularity. Also, in the event that I want to use or expose pieces of a system to blueprints, it would allow for a smooth transition. 

Since performance is a critical issue in this kind of simulation, I will default to not using blueprints and doing so selectively as desired.

### Outro

I go into more detail about my thoughts on and how to implement Components in C++ in UE <a href="{{ site.baseurl }}{% post_url R3HCSblog/2025-09-09-components-in-ue-c++ %}">here</a>

if you are interested.

See you in the next post,
<br>Cary