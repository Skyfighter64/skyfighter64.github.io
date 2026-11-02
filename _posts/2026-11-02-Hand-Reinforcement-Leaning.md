---
layout: post
title:  "Advanced Reinforcement Leaning | DIY Robot Hand"
date:   2026-11-02 00:13:19 +0200
categories: robots
project: robotics
---


## Introduction
The other day, I created a simulated version of a robotic hand from uni in order to try some reinforcement learning tasks using python and MuJoCo.

We already introduced the simulated robot hand and reinforcement learning [here]({{ "robots/2026/10/19/Robot-Hand-Simulation" | relative_url }}). This post covers some more advanced reinforcement learning tasks like catching a ball or rotating a cube.

You can find a video demonstration here:

<div class="video-container">
  <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/NnyTsH5burA?si=1lFoXSYl4txXqgWm" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

<style>
.video-container {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
}

.video-container iframe {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 0;
  border-radius: 8px;
}
</style>


## Trainer Concept
In order to perform more diverse reinforcement learning tasks, we once again used MuJoCo, OpenAI Gym, StableBaselines3 and PyTorch as the technology stack. Using these, we developed a modular training environment, which takes all the different components and combines them into a commandline-tool for training, testing and viewing our simulation tasks.

We define a task as something which contains:
- A __MuJoCo scene__ which contains the robot hand and all further 3D models 
- An __OpenAI Gym Environment__, which defines the accessible parameters of the MuJoCo scene and the reward function 
- A __"task"__ python class, combining the RL-Algorithm, environment and 3D model into one ready-to-go RL task

Given such a task, our RL-Trainer python script can perform different things, like
- Training, including :
  - detailed logging and tensorboard reward tracking 
  - Regular checkpoint evaluation 
  - Saving *best* and *latest* models
- Inference of the best and latest models
- Environment Inspection
- Further testing/debugging functions



## Proof-of-Concept: Touching Fingertips
This task was already covered in a previous post [here]({{ "robots/2026/10/19/Robot-Hand-Simulation" | relative_url }}). It instructs the robot hand to touch thumb and index finger by rewarding small distances between them. This task was mainly just for testing and finding bugs. 

## Task 1: Holding a Ball
The first real task contains the robot hand pointing upwards and a red ball laying on top of it. The task is simple: 

__Don't drop the ball__

To achieve this, our PPO agent gets rewarded for keeping the ball in a high position and punished every time the ball drops and rolls away. Furthermore we incentivize a firm grasp by explicitly rewarding contact between a finger tip and the ball. Lastly, as a general measure, we add a very small punishment for unnecessary movements.


<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/ppo_hold_ball_env.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The environment for the ball-holding task.</figcaption>
</figure>
</center>

Because of the big ball, our PPO agent found some rather interesting solution to this challenge: 
Instead of gripping firmly with all fingers, it opted to only use the outermost fingers and the thumb to clamp down the ball, while keeping the middle and ring fingers unused, potentially because they could interfere negatively with the ball and cause it to roll away.

We ran this experiment multiple times with different rewards, agents and environments, and found that the agent mostly opted for similar solutions every time. 

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/ppo_hold_ball_done.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>One of the solutions for this task. Note: This agent is different from the video.</figcaption>
</figure>
</center>

