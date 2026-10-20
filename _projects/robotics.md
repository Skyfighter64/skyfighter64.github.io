---
layout: post
title: Robotics Projects
description: A collection of my various robotics related posts
featured: true
---
<div class="image-gallery" style="--columns: 3;
            --gallery-width: 1100px;
            --image-height: 150px;">
  <img src="/media/robot/driving_angles.png" alt="Kinematic Architecture">
  <img src="/media/robot/mainboard_signal_injection.jpg" alt="Robot PCB">
  <img src="/media/robot_hand/real_robot_hand.jpg" alt="Robot Hand">
</div>

My robotic projects currently focus on an old vacuum robot which I modify for fun. I furthermore work with robot arms and hands to build advanced robot grippers at university and in my free time.



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