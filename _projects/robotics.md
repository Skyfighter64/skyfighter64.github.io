---
layout: post
title: Robotics Projects
description: A collection of my various robotics related posts
featured: true
---
<div class="image-row">
  <img src="/media/robot/driving_angles.png" alt="Kinematic Architecture">
  <img src="/media/robot/mainboard_signal_injection.jpg" alt="Robot PCB">
  <img src="/media/robot/robot.PNG" alt="Robot Vacuum">
</div>

My robotic projects currently focus on an old vacuum robot which I modify for fun. I furthermore work with robot arms to build advanced robot grippers in university, but have not found the time to write about these topics yet.



## Related Posts

<ul>
  {% for post in site.posts%}
    {% if post.project == page.slug %}
      <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
</ul>
(New to Old)