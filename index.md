---
title: "R3HCS Development Blog"
description: "Simulation, Unreal, and FEM Devlog"
theme: jekyll-theme-minimal
author: C
url: "https://caryrandazzo.github.io"
---

# Welcome to the R3HCS Development Blog
 Covering simulation, FEM, and plugin development in Unreal Engine.

This is the public record of development for the Realtime 3D Heat Conduction Simulator (R3HCS), including authorship, goals, and development milestones.

- [Portfolio](https://caryrandazzo.github.io/portfolio/)

## Featured Posts

- [Why I Rebuilt R3HCS](/2025/08/02/why-i-rebuilt-r3hcs.html)

After leading development on a real-time simulation system in a prior role, I decided to build an original version to push performance and ownership further...[continue](/2025/08/02/why-i-rebuilt-r3hcs.html)

## Recent Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <small>— {{ post.date | date: "%B %d, %Y" }}</small>
    </li>
  {% endfor %}
</ul>