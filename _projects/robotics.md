---
layout: post
title: Robotics
description: Collection of my robotics related posts
featured: true
---

## Related Posts

<ul>
  {% for post in site.posts %}
    {% if post.project == page.slug %}
      <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
</ul>