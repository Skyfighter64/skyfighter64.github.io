---
layout: post
title:  "Driving Around | Hacking a cleaning Robot"
date:   2024-10-21 23:51:33 +0200
categories: robot
---


*This post is part of a series about hacking and modding a cleaning robot.*


## Introduction

In my last post I described how I got a microcontroller to interact with the motor drivers on the robots mainboard. After that it was time to actually write some software so that we can use the wheels for more than just driving forward at full speed.

I want to use the robot for more demanding tasks such as automated driving in the future. For this to work, I need a bigger framework of all kinds of features and functionality, which is why I started to develop [Robocore](https://github.com/Skyfighter64/Robocore).

## The motor driver script
To set the speed and direction of each wheel using a simple function call like:

```cpp
//drive forward
leftWheel.Drive(speed=255)
//drive backwards
```
I needed some kind of script communicating with the motor drivers on the mainboard.


This would then be helpful for more abstract ways of driving like using inverse kinematics (more on that later).

The task for the motor driver script would be to drive one of the wheels in the following way:
- Drive the motor with a given speed
- Drive in reverse when a negative speed is given
- Be able to drive at slower than full speed (eg. 50% speed)
- Ignore invalid speeds (eg. 200% speed)
- Limit the motor to a minimum speed
- Do all this while using the two-wire signal inputs of the motor drivers on the mainboard



Realizing this was very straight forward. I only needed to provide the PWM-Signals corresponding to a wanted speed to the forward or backwards inputs of the motor driver and check if it's within certain boundaries. 

The minimum speed requirement mentioned above was needed since I found out that the motors were actually not turning for PWM-Signals below a certain threshold. This way I can just turn the motors of if such a speed is given, hopefully preventing them from getting fried.

For future use I also added some functions to retrieve the current speed and driving direction from the motor. 

The resulting code can be found in [motor_driver.cpp](https://github.com/Skyfighter64/Robocore/blob/main/src/core/motor_driver.cpp) inside my [Robocore](https://github.com/Skyfighter64/Robocore) project.


## Inverse Kinematics

//todo
