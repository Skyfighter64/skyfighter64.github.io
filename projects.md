---
layout: page
title: Projects
permalink: /projects/
---

<div class="image-row">
  <img src="/media/index/code.png" alt="Code Snippet">
  <img src="/media/alup/d1_mini_pcb.jpg" alt="PCB">
  <img src="/media/alup/leds_on.JPG" alt="ARGB LED Strip">
</div>


Let's face it, every computer scientist and engineer has a load of fun and interesting personal projects. While some of them stay small and unfinished, others have the potential to be something special.

Below are some blog posts of my biggest personal projects, which I deem to be worthy for others to see.
<br/>

---

<br/>



<div class="grid">
  {% for project in site.projects %}
    <div class="card">
      <h3><a href="{{ project.url }}">{{ project.title }}</a></h3>
      <p>{{ project.description }}</p>
    </div>
  {% endfor %}
</div>