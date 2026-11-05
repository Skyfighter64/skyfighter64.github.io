---
layout: post
title:  "Robot Arm Kinematic Learning | DIY Robot Hand"
date:   2026-11-05 00:14:16 +0200
categories: robots
project: robotics
---


## Introduction
I want to use my robot hand together with a Franka-Emika Panda robot arm. To achieve this, I decided to take on the challenge and teach a reinforcement learning agent to move a simulated robot arm.
<br/>
Specifically, I tried to teach the inverse kinematics of this 7 DoF arm to a Proximal Policy Optimization agent with the intention that my experience gained with this task will be valuable once we combine the arm with the robot hand to execute more complex tasks.

## Forward and Inverse Kinematics
The kinematics of a robot describe its movement model based on the available joints, usually as a mathematical formula. This is traditionally calculated manually using for example the [Denavit Hartenberg Convention](https://en.wikipedia.org/wiki/Denavit%E2%80%93Hartenberg_parameters).
<br/>
<br/>
The __forward kinematics__ of a robot arm describe at which position and rotation the end effector (the 'gripper') ends up in when the robot is given a specific set of joint angles (a 'configuration'). More practically, for a robot arm like the Franka-Emika Panda with a total of 7 rotation joints, the forward kinematics would describe where the end effector ends up given an angle for each joint.
<br/>
<br/>
The __inverse kinematics__ on the other hand describe the exact opposite: Given a position and rotation in the real world, the inverse kinematics describe, which angles each robot joint needs to have in order to reach this pose. 
<br/>
<br/>
If you think about it, this is very useful because we can just tell the robot where to move and the inverse kinematics would calculate the needed movements. However, this is also considered to be more difficult than the forward kinematics, because robot arms with many joints, like the Panda, have multiple joint configurations for the same end effector pose, where the rest of the arm is positioned differently.
<br/>
<br/>
To visualize this, think of it like your own arm: When reaching for a bottle of water on a table in front of you, you could do so with your elbow pointing straight downwards, or also while your arm is angled with the elbow to the side. You can therefore reach the same position with two slightly different angles in your arm. 
<br/>
<br/>
In real-life this does not really matter for you, but when controlling a robot arm it needs to be decided which of the possible options is chosen in order to resolve this ambiguity, because mathematic formulas usually don't understand a "I don't care".
<br/>
<br/>
And this exactly is the appeal in learning the kinematics using a reinforcement learning agent. Similar to a human, it could learn, that it is mostly irrelevant in which configuration the robot arm reaches a pose, and could choose the one for which it would receive the most reward or least punishment (Also, I'm too lazy to calculate the kinematics manually xD).



## Environment & Setup
For the robot arm in question, we simulate the Franka-Emika Panda in MuJoCo, using the `mjcf` models provided by [Google Deepmind on GitHub](https://github.com/google-deepmind/mujoco_menagerie/tree/main/franka_emika_panda). 
<br/>
Using this model, we created a simple scene with the panda arm and a target, shown as a coordinate origin with X, Y and Z axes. The position and rotation of this target randomized position and rotation within a displayed working area.
<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/panda_reach_pose_env.png" alt="Image of the robot arm " width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The robot arm together with the randomized target pose.</figcaption>
</figure>
</center>

# The Task
The task which the RL agent has to fulfill in order to learn the inverse kinematics is straight forward:

- Given a random target, position the end effector in the same position and orientation as said target.
- While doing so, don't collide with anything
- Also minimize unnecessary fast movements

This was implemented into the reward function in the following way:
- Give more reward to the agent the closer the end effector is to the target, both in position and rotation
- Incentivize fast learning by rewarding any movement towards the target and punish movements away from it

- Additionally, we consider the task as solved, if the end effector is within 3cm and 20 degrees of the target pose. These values were lowered once the agent proved its capability to learn the kinematics successfully.


## Results

After a total of TODO time steps, the PPO agent successfully learned to approximate the kinematics of the Panda robot arm, with a total training time of TODO minutes.

Here's a quick video of the final solution:



<!-- TODO Add image -->
<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/panda_reach_pose_env.png" alt="Image of the robot arm " width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The agent with the inverse kinematic solved.</figcaption>
</figure>
</center>


## Lessons Learned
In total, it took over a week of work to get this finally working. During this time, many completely different rewards, observations and action strategies were tested and debugged, so here are some 'lessons learned':

1. As always with RL, make sure the training code is bug-free. Also make sure that reward functions are doing what they are supposed to do. Check so manually, especially watch the signs and directions of the reward (swapping +- is a common issue).
2. For this Panda MJCF specifically, make sure to copy the simulation parameters from the robot arm model into the final MJCF scene. Otherwise the robot arm kinematics cannot be learned because the simulation is sluggish.
3. When using absolute actions, observations and rewards (eg. absolute joint positions instead of increments for the agent action), make sure the agent gets AT LEAST one full episode of experience for every training run. For PPO, this is achieved by choosing `n_steps` larger than the maximum number of steps in one episode. Learning on half-episodes worked very poorly
4. Exponential reward functions like `reward = np.exp(-2.0 * distance)` work very well for distances and differences. They keep the reward between 0 and 1 with exponential increments the better it gets (could this be because of the derivation property of the exponential functions?).
5. Plot rewards and reward composition to make debugging rewards easier. This helps when testing if the task itself is even possible.



<!-- TODO 
### Video Demonstration




<div class="video-container">
 <iframe width="560" height="315" src="" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
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
<br/>
