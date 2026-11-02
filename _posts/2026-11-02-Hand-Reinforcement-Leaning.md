---
layout: post
title:  "Advanced Reinforcement Leaning | DIY Robot Hand"
date:   2026-11-02 00:13:19 +0200
categories: robots
project: robotics
---


## Introduction
The other day, I created a simulated version of a robotic hand from uni in order to try some reinforcement learning tasks using python and MuJoCo.

We already introduced the simulated robot hand [here]({{ "robots/2026/10/19/Robot-Hand-Simulation" | relative_url }}). This post covers some more advanced reinforcement learning tasks like catching a ball or rotating a cube.

### Video Demonstration
Showing the acutal tasks using only images is rather difficult, so here is a demonstration video of the upcoming tasks:

<div class="video-container">
 <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/EZHX6HrrSx8?si=8FgArtpAKQSaRddP" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
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

## Software and Trainer Concept
In order to perform more diverse reinforcement learning tasks, we once again used MuJoCo, OpenAI Gym, StableBaselines3 and PyTorch as the technology stack. Using these, we developed a modular training environment, which takes all the different components and combines them into a commandline-tool for training, testing and viewing our simulation tasks.

We define a task as something which contains:
- A __MuJoCo scene__ which contains the robot hand and all further 3D models 
- An __OpenAI Gym Environment__, which defines the accessible parameters of the MuJoCo scene and the reward function 
- A __"task"__ python class, combining the RL-algorithm, environment and 3D model into one ready-to-go RL task

Given such a task, our RL-trainer python script can perform different things, like:
- Training, including:
  - Detailed logging and tensorboard reward tracking 
  - Regular checkpoint evaluation 
  - Saving *best* and *latest* models
- Inference of the best and latest models
- Environment Inspection
- Further testing/debugging functions



## Proof-of-Concept: Touching Fingertips
In this task, the robot hand should move its fingers with the goal to touch the thumb and index finger tips.
This was achieved by rewarding small distances between the two white dots on each finger. 

Note that this task was mainly just for testing and finding bugs. It was already covered in a previous post [here]({{ "robots/2026/10/19/Robot-Hand-Simulation" | relative_url }}).

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/finger_pinch_blue.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>A solution for this task found by the agent.</figcaption>
</figure>
</center>

## Task 1: Holding a Ball
The first real task contains the robot hand pointing upwards and a red ball laying on top of it. The task is simple: 

- __Don't drop the ball__

To achieve this, our PPO agent gets rewarded for keeping the ball in a high position and punished every time the ball drops and rolls away. Furthermore we incentivize a firm grasp by explicitly rewarding contact between a finger tip and the ball. Lastly, as a general measure, we add a very small punishment for unnecessary movements.


<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/ppo_hold_ball_env.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The environment for the ball-holding task.</figcaption>
</figure>
</center>

Because of the big ball, our PPO agent found some rather interesting solutions to this challenge: 
Instead of gripping firmly with all fingers, it opted to only use the outermost fingers and the thumb to clamp down the ball, while keeping the middle and ring fingers unused, potentially because they could interfere negatively with the ball and cause it to roll away.

We ran this experiment multiple times with different rewards, agents and environments, and found that the agent mostly opted for similar solutions every time. 

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/ppo_hold_ball_done.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>One of the solutions for this task. Note: This agent is different from the video.</figcaption>
</figure>
</center>



## Task 2: Catch Falling Ball
This second task is a more advanced version of the previous one: The robot hand now points downwards with the ball spawning just below. After one second of simulation-time, the ball starts falling down. The goal onf this task is once again simply: 

- __Don't drop the ball__

Similarly to before, our PPO agent gets rewarded for keeping the ball in a high position and punished every time the ball drops. In this case however, we needed to shrink the ball a little bit because holding the big one from before was not really possible due to reach limitations.


<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/catch_ball_task.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The environment for the ball-catching task.</figcaption>
</figure>
</center>

After some training, the agent successfully used the hovering time of the ball to grab it right before it was able to fall down. It did so by pinching it between the thumb and index finger while curling up the ring finger to further support the ball and secure it against sideways movements. This was surprisingly similar to how a human could handle such a situation, assuming he would not be able to reposition its hand in any other way.
<figure>
<div class="image-gallery" style="--columns: 2;
            --gallery-width: 1100px;
            --image-height: 300px;">
  <img src="{{site.baseurl}}/media/robot_hand/catch_ball_solve.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <img src="{{site.baseurl}}/media/robot_hand/catch_ball_solve_2.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>

</div>
<center>
  <figcaption>A solution for this task found by the agent.</figcaption>
  </center>
</figure>

<!-- <center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/catch_ball_solve.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>A solution for this task found by the agent.</figcaption>
</figure>
</center> -->



## Final Task: Rotate a Dice
With the third and last task, we wanted to explore the limits of our 8 DoF robot hand. Other (more expensive) hands with many more degrees of freedom are able to rotate a cube to a desired position within their own palm. Because our hand is missing some of these movements, we simplified the task a little and placed the cube below the hand on the ground. With that, the goal for the final task is: 

- __Rotate the cube into a given position__


Specifically, we created a random rotation around the z-axis and tasked the agent to rotate the cube into this exact position. Like before, it was also slightly punished for unnecessary movements.
We used a Soft Actor Critic agent for this task because they are known to be more sample-efficient than PPO.

<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/rotate_cube_env.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>The environment for the dice-rotation task.</figcaption>
</figure>
</center>

Our SAC agent managed to solve this task with varying degrees of success. It mostly was able to rotate the cube roughly into the same orientation as the displayed reference, with some situations being very accurate and others roughly correct. 

Some orientations however were more difficult to reproduce for the agent: <br/>
Sometimes, the cube would move out of reach, which meant the agent was unable to further rotate it. These situations either resulted in the agent still trying, but kicking the cube away, or it refused to touch the cube and remained in an incorrect, but seemingly "could be worse" position. 

This behavior was somewhat expected due to the limited motions of the robot hand considering the shared finger adduction over all four fingers, and the underactuateded open/close movements of each finger using only one single actuator.


<center>
<figure>
  <img src="{{site.baseurl}}/media/robot_hand/rotate_cube_solved.png" alt="Image of the robot hand" width="60%" style="display: block; margin: 0 auto" class="rounded-image"/>
  <figcaption>A solution for this task found by the SAC agent.</figcaption>
</figure>
</center>


Over all described tasks, we found that our robot hand could solve a surprising amount of them, even partially solving the more difficult cube rotation task which showed the limits of 8 DoF robot hands. We once again noticed that reinforcement learning tasks heavily rely on good reward function structure and using a clear target with guiding incentives and punishments.

In the future, we hope to test these tasks on the real version and learn more about the challenges when applying simulated RL tasks to the real world.