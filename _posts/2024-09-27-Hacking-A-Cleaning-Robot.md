---
layout: post
title:  "Hacking a cleaning Robot"
date:   2024-09-23 00:02:54 +0200
categories: test
---


## Introduction
The other day I got my hands onto "Bruneau Kliver TP-AVC710" cleaning robot and wanted to make it drive autonomously using a Microcontroller. After opening it up and looking at the rather
 simple PCB, I thought it might be possible to inject signals directly into it, eliminating the need for new motor controllers 
 and a battery charging circuit.

 That's why I started analyzing the contents of the PCB and traced all copper lines and parts to its original purpose.

 Rainer Rebhan also hacked this robot, if you don't want to keep the old PCB is suggest checking out [his website](http://www.rainer-rebhan.de/proj_saugrob.html#).

## Robot Overview

The robot itself is rather simple. It drives around in fixed
 patterns, reversing and turning away if it hits something with its front bumper.
It is turned on using a switch at the top and can be charged using an external 14.4V Power Supply. 

Todo: picture here

## Main PCB Features
The default PCB already comes with most of the needed features:
- Battery Charging/Discharging logic
- Motor Driver for both wheel motors
- On/Off controll for the vacuum motor
- Collision Detection using the front bumper
- Two DC/DC Converters


## PCB Sections
<img src="{{site.baseurl}}/media/robot/main_pcb.PNG" alt="Main PCB Blocks" width="1000"/>

### 6-Pin Connector
The 6 Pin JST on the right side connects the main PCB to two small daughterboards in the casing. These connect to the battery, PSU and ON/OFF switch. This also connects the vacuum motor.

| Pin | Purpose                               |
|-----|---------------------------------------|
| 1   | V_battery (Positive battery terminal) |
| 2   | V_psu (Positive Powersupply termial)  |
| 3   | Power Switch (ON: V_Battery, OFF: NC) |
| 4   | Ground                                |
| 5   | Vacuum Motor +                        |
| 6   | Vacuum Motor -                        |

Note that the Power Switch at Pin 3 is connected to V_battery if ON. This serves as the main power source for the robot when running. Pin 1 on the other hand is used to charge the robot when a PSU is connected. 


### Main IC

The main IC seems to be either a microcontroller or (more likely) a custom ASIC. It controlls basically everything on the robot including:
- Both driving Motor's speed
- Vacuum Motor ON/OFF
- Battery charging
- Blinky LEDs
- Collision Detection


<img src="{{site.baseurl}}/media/robot/main_ic.PNG" alt="Pinout for the main IC" width="800"/>


#### Pinout:

| Mode   | Purpose             | Pin |   |   |   |     |   |   |   | Pin | Purpose                 | Mode   |
|--------|---------------------|:---:|--:|---|---|:---:|---|--:|---|:---:|-------------------------|--------|
|        |                     |     |   | ┌ | ─ | \\_/ | ─ | ┐ |   |     |                         |        |
| Output | Green LED           | 1   | ─ | ┤ | 0 |     |   | ├ | ─ |  14 | Red LED                 | Output |
| Input  | Light Barrier       | 2   | ─ | ┤ |   |     |   | ├ | ─ |  13 | Right Wheel Forward     | Output |
|        | NC                  | 3   | ─ | ┤ |   |     |   | ├ | ─ |  12 | Right Wheel Reverse     | Output |
|        | V_logic (5V)        | 4   | ─ | ┤ |   |     |   | ├ | ─ |  11 | Ground                  |        |
| Input  | Comparator          | 5   | ─ | ┤ |   |     |   | ├ | ─ |  10 | Battery Charging Enable | Output |
| Output | Vacuum Motor On/OFF | 6   | ─ | ┤ |   |     |   | ├ | ─ |  9  | Left Wheel Reverse      | Output |
| Input  | Sense PSU Presence  | 7   | ─ | ┤ |   |     |   | ├ | ─ |  8  | Left Wheel Forward      | Output |
|        |                     |     |   | └ | ─ | ─   | ─ | ┘ |   |     |                         |        |


### Motor Drivers
There are two motor drivers, each consisting of a H-Bridge and some upstream logic.

The following pins from the main IC are used for driving:

| Pin NR | Motor | Purpose |
|--------|-------|---------|
| 13     | Right | Forward |
| 12     | Right | Reverse |
| 9      | Left  | Reverse |
| 8      | Left  | Forward |

The previously mentioned extra logic makes sure that the robot does not try to
drive forward and reverse at the same time:

| Pin 8 \| 13 (Forward) | Pin 9 \| 12 (Reverse) | Motor Driver Output |
|-----------------------|-----------------------|---------------------|
| 0                     | 0                     | Stop                |
| 0                     | 1                     | Backwards           |
| 1                     | 0                     | Forward             |
| 1                     | 1                     | Backwards           |


## Charging Circuit
todo

### Light Barrier
The light barrier is used for detecting collisions using the front bumper. If the robot bumps into something while driving, a plastic piece blocks the light barrier and the robot stops, reverses and turns around.


## Power Distribution
The PCB has two main Power Sources: The Battery and an external Power Supply. 
From these, two different DC/DC Converters create the voltages needed by the components and motors.

Here is a quick overview over the different power rails:
<img src="{{site.baseurl}}/media/robot/power_rails.PNG" alt="PCB power rails overview" width="1000"/>
Note that the Image is mirrored to make it easier matching the components on the other side.

### L7805CV Voltage Regulator
This Voltage regulator receives the variable voltage from either the Battery or the Power Supply (if connected) and regulates it down to a constant 5.02V. It is used to power the main IC and the light barrier.

According to its data sheet, it has all kinds of protection circuits and can provide up to 1.5A of current (although this requires an additional heat sink).


### 34063AP1 Buck/Boost Converter
This is the second DC/DC Converter on this board. Its purpose is to provide 8.34V to the motor controller of both driving motors.
