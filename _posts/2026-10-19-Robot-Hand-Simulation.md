---
layout: post
title:  "Robot Hand Simulation | DIY Robot Hand"
date:   2026-10-19 00:15:19 +0200
categories: robots
project: robotics
---


## Introduction
The other day, I created a simulated version of a robotic hand from uni in order to try some reinforcement learning tasks using python and MuJoCo.

## Robot Hand Overview
The robot hand itself is a 3D printed DIY-hand which was designed and built during a thesis at my university.
With a total of 8 Degrees of Freedom and 3D printed joints, it is meant to be inexpensive and easy to build, while being able to grasp simple things with sufficient force.

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/real_robot_hand.jpg" alt="Image of the robot hand" width="40%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The real robot hand, 3D printed in transparent blue PLA.</figcaption>
</figure>
</center>

The available movements include opening and closing each finger separately and spreading all fingers at once.
Furthermore, the thumb is capable of moving in a total of 3 degrees: 
- Thumb flexion, aka. opening and closing the thumb
- Radial adduction, aka. left/right movement
- Palmar adduction, aka. rotating around the palm

This enables the robot hand to grab things like wooden blocks, a ball, or a screw driver and perform tasks using teleoperation or record-and-replay actions. 

## The Task
Because robot hand hardware is very cool, but rather useless without any software, I decided to use the current developments in artificial intelligence to my advantage
by trying to apply some Reinforcement Learning algorithms to the robot hand. 

For this, I decided to start with a simple control task: 

Make the robot hand touch the tips of the thumb and index finger at a specific position.
While doing so, the agent controls the simulated motors of the virtual hand and gets the current position of 
each motor as feedback, just like the real-deal.

## Simulating the Robot Hand
In order to train a Reinforcement Learning agent, we couldn't just use the real hand, because it would be way too slow and expensive,
so an accurate virtual simulation model was needed.
We decided on using [MuJoCo](https://mujoco.org/) as the simulation environment because it can be easily combined with OpenAI's Gymnasium and StableBaselines3, which we will use later for the actual RL-algorithms.

To simulate our robot hand in MuJoCo, we used URDF and later MJCF files, which contain an XML-like description of the robot, including linkages, joints, motors and even tendons. 
Unfortunately, we needed to manually create these files from the design files of the real robot hand because the available conversion tools could not handle our complex CAD design.

By combining over 40 3D objects using virtual joints, tendons, and actuators, we therefore accurately recreated the capabilities and movements of the real hand.
Furthermore, we needed to control the virtual clone in the same way as the real hand, so a software compatibility layer was developed, which takes the robot hand movement commands and converts them into 
positional commands for the virtual actuator. This ensures that we are able to test the finished reinforcement learning task on the real robot hand in the future and not just have a simulation result with no real purpose.  

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/virtual_robot_hand.png" alt="Image of the virtual robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>Simulation of the virtual robot hand in MuJoCo.</figcaption>
</figure>
</center>

## Reinforcement Leaning
In Reinforcement Learning, a so called 'agent' (like a player) learns to fulfill a task by receiving rewards whenever he does something well, and punishment whenever something bad happens (don't worry, it does not hurt xD).
This happens inside a simulated environment, just like a game: 

1. The agent observes the environment and chooses to do an 'action', eg. move a motor or something similar.
2. The simulation then runs for a bit and gives the agent feedback if this action resulted in something good or bad (the reward mentioned before)    
3. This happens multiple times, performing a series of actions and collecting many rewards.
4. The agent then takes all the collected rewards and trains an internal neural network to learn from its actions.

This process repeats over-and-over until the agent (hopefully) gets very good at this task.

For our case, we chose a PPO-Agent from [StableBaselines3](https://stable-baselines3.readthedocs.io/en/master/) for testing purposes, which is one of the easier and more stable Reinforcement Learning algorithms.
To define the exact touching locations, we placed two white dots at the finger tip of each finger and the thumb. The agent was then tasked to make these dots on the index finger and the tip of the thumb touch, which was realized by rewarding the agent depending on how close the dots are. The other fingers had no task and could do whatever they want (mostly just random movements xD). 

## Some Tips:
This was not our first time training a reinforcement learning agent, however, we still did a lot of things wrong and learned a lot, so here are some tips if you want to try something similar yourself:
1. __Make sure everything else works!__<br/>
This is the most common error source. If there are bugs in your code surrounding the RL-Task, the learning will just not work and be very frustrating.<br/>
There is no way to tell whenever something is buggy or not during the RL-Training, except that the agent will just never learn the task. Therefore, it is very important to thoroughly test every step of your 
simulation environment and conversion functions before going all-in on the learning.

2. __Start with a trivial task.__<br/>
Starting with a complicated or challenging task often results in complete chaos even for the smallest bugs.
Therefore, pick something so trivial that you could do it yourself and verify that this is easily possible by completing the task manually or using traditional software.<br/>
In our case, our debug-task was to move one finger of the hand into a specific position, which we could also trivially do without RL by just commanding the robot motors to move there.


3. __Use defaults first!__<br/>
Use recommended default settings and values for your RL-Algorithm first!
Most of the time, this results in the agent learning very slow, however you NEED a starting point to verify that the agent can actually learn the task.<br/>
From there, you can start adjusting different values like the learn rate, maximum epochs, number of experiences, and so on, in order to speed up future learning.  

4. __Watch your agent play.__<br/>
This is either very fun or very frustrating.<br/>
Watch your agent during or after training to verify it is actually doing what it's supposed to do. 
To do so, watch it visually in the simulation and log the received rewards during training (eg. using [Tensorboard](https://www.tensorflow.org/tensorboard)).<br/>
This is an easy way to spot bugs or conceptual errors in your task, and observe if the agent actually improves or stagnates.

5. __Let the simulation settle.__<br/>
In case your simulation includes time-delayed physics like springs or motor ramp-up and ramp-down, the simulation itself usually needs to run for more than one simulation step in-between each agent action.<br/>
Only this way the simulated physics can actually adjust to a given action and settle down before the next action arrives. 

## Final Results

After a lot of debugging and testing, our PPO agent needed about 3 minutes or `100.000` training steps before successfully touching the index finger and thumb tips for the first time.
It was then able to repeat this action using deterministic policy upon request (still a little shaky though). 

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/index_pinch_finished.png" alt="The two white dots at the index finger and thumb touching each other" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>Resulting pose of our RL policy.</figcaption>
</figure>
</center>

We also tested touching the other fingers with the thumb, all of which worked in the simulation. For challenging cases due to the limited range of motion, like the thumb and ring finger, it found an interesting approach to this more difficult pose.

<div class="image-gallery" style="--columns: 3;
            --gallery-width: 1100px;
            --image-height: 200px;">
  <img src="{{site.baseurl}}/media/robot_hand/ring_finger_pinch.png" alt="Image 2" class="rounded-image">
  <img src="{{site.baseurl}}/media/robot_hand/ring_finger_pinch_2.png" alt="Image 3" class="rounded-image">
  <img src="{{site.baseurl}}/media/robot_hand/ring_finger_pinch_3.png" alt="Image 4" class="rounded-image">
</div>

<!-- Here's a quick video of how the training looked like:
(TODO) -->

I personally would really like to see if our task also works on the real robot hand, but sadly we currently don't have access to the real hardware, so this might be something for the future.