---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---
![Test Image](media/index/nordseher-castle-crop.JPG)
<br/>
Image Credits: [Nordseher on Pixabay](https://pixabay.com/de/photos/burg-architektur-schiff-nebelmeer-8519077/)
<br/>
# Welcome to my homepage :)
---
<br/>
I am a university student, currently doing my master's degree in computer science.
This page is all about my personal projects, ideas, hobbies and interests. Here you can find the newest and latest infos and enjoy some computer science, electrical engineering and technology-related contents.


__<u>Professional Interests:</u>__ \
To bridge the barrier between hardware and software, I am creating optimized solutions using my skills in coding, 3D printing and physics in the areas of:

- Mobile Robots, Robot Arms and Robotic Hands
- Microcontrollers and Embedded Devices
- Communication and Networking

__<u>My personal interest and hobbies:</u>__
- Working on my Robotics and Embedded projects
- Lighting up the world with my addressable LEDs
- Repairing and fixing everyday items
- CAD design, 3D-Printing and 3D printer hardware
- Photography
- and more...


# Featured Projects
<div class="grid">
  {% assign featured_projects = site.projects | where: "featured", true %}
  {% for project in featured_projects %}
    <div class="card">
      <h3>
        <a href="{{ project.url }}">{{ project.title }}</a>
      </h3>
      <p>{{ project.description }}</p>
    </div>
  {% endfor %}
</div>


<br/>

---
# Social Links: 
- [GitHub](https://github.com/Skyfighter64)
- [iFixit](https://www.ifixit.com/User/Contributions/4374656)
- [Printables](https://www.printables.com/@Skyfighter)

---
