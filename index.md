---
title: "R3HCS Development Blog"
description: "Simulation, Unreal, and FEM Devlog"
theme: jekyll-theme-minimal
author: C
url: "https://caryrandazzo.github.io/R3HCSblog"
---

# Welcome to the R3HCS Development Blog
 Covering simulation, FEM, and plugin development in Unreal Engine.

This is the public record of development for the Realtime 3D Heat Conduction Simulator (R3HCS) including goals and development milestones.

## Featured Posts

- [Why I Rebuilt R3HCS](/R3HCSblog/2025/08/02/why-i-rebuilt-r3hcs.html)

After leading development on a real-time simulation system in a prior role, I decided to build an original version to push my skills further as well as the performance of the system while experimenting with a variety of different approaches...[continue](/R3HCSblog/2025/08/02/why-i-rebuilt-r3hcs.html)

## Recent Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
      <small>— {{ post.date | date: "%B %d, %Y" }}</small>
    </li>
  {% endfor %}
</ul>