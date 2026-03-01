---
layout: page
title: Projects
permalink: /projects/
---

This page serves as an overview over my projects.

---
<div class="grid">
  {% for project in site.projects %}
    <div class="card">
      <h3><a href="{{ project.url }}">{{ project.title }}</a></h3>
      <p>{{ project.description }}</p>
    </div>
  {% endfor %}
</div>