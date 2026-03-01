---
layout: post
title:  "Time Synchronization for the ALUP Protocol"
date:   2025-09-10 00:14:49 +0200
categories: timesync
project: alup
---

*This post is part of a series about my [ALUP Protocol](https://github.com/Skyfighter64/ALUP) side project.*


# Introduction

The 'ALUP' is an application layer protocol to communicate RGB-Data from a PC to a Microcontroller in order to light up addressable LED strips from devices with more computing power than an Arduino.

Originally it was only intended for the use with one single ALUP Receiver, but more recently I figured that it would be useful if multiple Receivers could be grouped together to update their LEDs synchronously. More specifically, I wanted to add timestamps to each Data Frame, which then tells the Microcontroller when exactly to update the LEDs. This requires time synchronization for both devices.

# Background
A computer usually synchronizes it's local time with accurate time servers over the internet. This makes its internal timekeeping sufficiently exact for everyday use. Most Microcontrollers however do not support this feature and are often not even connected to the internet at all. For them, time tracking is done by an internal timer which starts at 0 when they first boot up. 

These cheap internal timers are often prone to some drift, where the internal clock counts either slightly faster or slower than the real time (usually ~ $3-4 \frac{s}{day}$).
<!---
In order to synchronize the time between a PC and a Microcontroller, we need to:
- Calculate the Offset of the Microcontroller's clock to the real time
- Update the offset  often to minimize the inaccuracy through drift 
- Integrate it into the existing protocol 
- Make it work efficiently
-->
## The generic Precision Time Protocol's Synchronization Mechanism

About half a year ago I was first introduced to the time synchronization mechanism of the generic Precision Time Protocol in university. It was fascinating for me how simple it was and how well it worked once I understood the principle behind it.

<figure>
  <img src="https://upload.wikimedia.org/wikipedia/commons/4/48/Grundlegender_PTP-Nachrichtenaustausch.png" alt="PTP Time Synchronization Mechanism"/>
  <figcaption>The gPTP Time Synchronization Mechanism (src: 
<a href="https://de.wikipedia.org/wiki/Precision_Time_Protocol#/media/Datei:Grundlegender_PTP-Nachrichtenaustausch.png">Wikipedia: PTP (German)</a>)</figcaption>
</figure>

The setup is similar to our Microcontroller situation: A Master device which has the accurate time (master time domain), and a Slave device which has its own local time (slave time domain) synchronize their time using time measurements.

How it works:
1. The master device sends its accurate time $T$ to the slave. Because of the unknown transmission delay ( 10-100ms or more), the received time $T$ is outdated once it reaches the slave. For this reason, the Master notes the time it sent out the packet ($t_1$) and the sender notes the time it received the packet ($t_2$).
2. The master transfers $t_1$ to the Slave in a follow up packet.
3. The slave sends a packet to the master and notes the sending time $t_3$.
4. The master notes the receiving time $t_4$ and responds to the delay packet by sending $t_4$ to the slave.

The slave now has all time stamps $t_1, t_2, t_3, t_4$ and can calculate the transmission delay $\Delta_t$:

$$\Delta_t = \frac{(t_2-t_1) + (t_4-t_3)}{2}$$

With this, he can correct the previously exchanged Time $T$:

$$T_{corrected} = T + \Delta_t$$

### Assumptions and Caveats
This method of time-correction makes some non-trivial assumptions:
- There should be no clock drift on either side 
- Transmission latency of $T$ needs to be the the same as $\Delta_t$

Which implies that:
- Sending and receiving latencies are the same (symmetric)
- Sending latency and $T$'s transmission latency are the same

These assumptions are the source of error in practical applications. Especially when working with wireless connections, the latency spikes caused by the layer 1 backoff-algorithms, like CSMA/CA in WiFi, can lead to inaccuracies. For this reason, we found that it is good practice to make multiple time synchronizations and take the median as a final result. 

To reduce the impact of time drift, it is furthermore recommend to frequently repeat the synchronization Algorithm as often as needed.

### Some further Notes:
In conclusion, all that is done is:
1. Transmit the accurate time $T$ from the master to the slave
2. Measure the transmission delay and correct $T$ on the slave

Simple, right?

Also, one might think: "If the sending and receiving latencies need to be symmetric, why do we measure both and not only the sending latency?".


It's because we actually **need** the measurement in both directions due to the of the different time domains on the master and slave before synchronization (aka. we need to consider which clock measured which  time).

We can't just subtract $t_2$ and $t_1$ because $t_1$ is in the master time domain and $t_2$ is in the slave time domain (The same goes for $t_4$ and $t_3$). 


However, subtracting $t_4$ and $t_1$ as well as $t_3$ and $t_2$ is entirely valid, because the first two were both measured by the master's clock and the second two by the slave's. Calculating their difference therefore works just fine and eliminates the constant offset so we can calculate the average.

In formulas, it therefore would make more sense to rewrite:

$$\Delta_t = \frac{(t_2-t_1) + (t_4-t_3)}{2}$$

as:

$$\Delta_t = \frac{(t_4-t_1)-(t_3-t_2)}{2}$$


# Adaptation for ALUP
With the ALUP being structured very differently, we made some adaptations in order for this mechanism to work:
1. The protocol has only two communication steps (see [ALUP Docs](https://github.com/Skyfighter64/ALUP/blob/master/Documentation/Documentation_en-us.md#data-transmission)), therefore, we  don't want to do the 4-step handshake from gPTP but rather simplify it to two steps.
2. ALUP tries to put as much work on the Sender (gPTPs Master) as possible, therefore we want to track both times on the Sender instead of telling the Receiver (Slave) to correct its own time.

So the overall goals are:

1. Synchronize the Receiver time as often as possible
2. Reduce the four steps from gPTP to only  two
3. Make the Sender (Master) keep the times instead of the Receiver (Slave)

<!---
1. Synchronize the Receiver time with every ALUP Packet
2. Track the time offset $\delta^S_R$ for all ALUP Receivers on the sender
3. When sending a packet: Convert any timestamps (internally) from Sender's local time to the respective Receiver's local time
-->
For this, we needed to turn around the gPTP's mechanism so that the Sender (Master) has $t_1,t_2,t_3,t_4$ in the end. This step also already reduces the communication effort from 4 step to only two (yay :>).
The new communication diagram looks something like this:

<center>
<figure>
  <img src="{{site.baseurl}}/media/alup/time_sync.PNG" alt="Reduced time synchronization graph" width="60%" style="display: block; margin: 0 auto"/>
  <figcaption>The simplified time synchronization graph</figcaption>
</figure>
</center>

With:

$$\Delta_t = \frac{(t_2-t_1) + (t_4-t_3)}{2}$$


The delta is still calculated in the same way as before, but you may notice that we do not send $T$ over to the Sender. This is because we don't need a specific time from the Sender it but can rather use any other time such as $t_3$ for a replacement. This simplifies the communication further

from: 

$$T_{Receiver} = T + \Delta_t$$

to: 

$$T_{Receiver} = t_3 + \Delta_t$$

Therefore, we now got the receiver's local time $T_{Receiver}$ successfully from the receiver to the sender with minimal error thanks to $\Delta_t$.

Since it is more practical, instead of storing $T_{Receiver}$ we rather store the offset for the Receiver's time to the Sender's time $\delta^S_R$: 

$$\delta^S_R = T_{Receiver} - T_{Sender} $$

**Note:** For $\delta^S_R$, the $S$ and $R$ denote that this is the offset of the **S**ender's time and the **R**eceiver's time.

This way, we don't need to track the Receiver's time on the Sender separately, but can instead use the Sender's system time at any point to calculate the corresponding local time on the Receiver by using:

$$T_{Receiver}  = T_{Sender} + \delta^S_R $$

The last simplification we do is to choose the measuring point for  $T_{Sender}$ smartly. To be more specific, we choose $T_{Sender}$ to be the same as $t_4$.

### Simplification of the formula

From before, we found the following formulas:

1. $\Delta_t = \frac{(t_2-t_1) + (t_4-t_3)}{2}$
2. $T_{Receiver} = t_3 + \Delta_t$
3. $\delta^S_R = T_{Receiver} - T_{Sender} $
4. $T_{sender} = t_4$

We combine and simplify them with respect to $\delta^S_R$ in order to get one singular formula which is simpler to write and easier to calculate:

- Start with:

$$\delta^S_R = T_{Receiver} - T_{Sender} $$

- Put in $T_{Receiver} = t_3 + \Delta_t$:

$$ \delta^S_R = (t_3 + \Delta_t) - T_{Sender} $$

- Put in  $T_{Sender} = t_4$:

$$\delta^S_R =  (t_3 + \Delta_t) - t_4$$

- Put in $\Delta_t = \frac{(t_2-t_1) + (t_4-t_3)}{2}$:

$$\delta^S_R = (t_3 + \frac{(t_2-t_1) + (t_4-t_3)}{2}) - t_4$$

- Rewrite subtractions without brackets:

$$\delta^S_R = \frac{(t_2-t_1) + (t_4-t_3)}{2} - t_4 + t_3$$

- Move $t_4, t_3$ onto the fraction:

$$\delta^S_R = \frac{(t_2-t_1) + (t_4-t_3)}{2} +\frac{-2\cdot t_4}{2} + \frac{2\cdot t_3}{2}$$

$$\delta^S_R = - \frac{(t_2-t_1) + (t_4-t_3) - (2\cdot t_4) + (2\cdot t_3)}{2}$$

- Make it pretty and simplify:

$$\delta^S_R = - \frac{t_2-t_1 + t_4-t_3 - 2\cdot t_4 + 2\cdot t_3}{2}$$

$$\delta^S_R = - \frac{t_2-t_1 + \cancel{t_4}-\cancel{t_3} - \cancel{2}\cdot t_4 + \cancel{2}\cdot t_3}{2}$$

$$\rightarrow \delta^S_R = - \frac{t_2 - t_1 - t_4 + t_3}{2}$$

Or:

$$\rightarrow \delta^S_R = - \frac{- t_1+ t_2 + t_3 - t_4 }{2}$$

This formula can calculate the offset from the Receiver's time to the Senders Time very accurately. We can use it to get the Receiver's local time from the Sender's local time very accurately:

$$T_{Receiver} = T_{Sender} + \delta^S_R $$

Note though that the caveats and assumptions from gPTP still apply, therefore:
- Timer drift is not considered -> synchronize time often (We do it with every packet)
- Synchronization error increases if the sending and receiving latency are not the same -> Collect multiple time synchronizations and use a running median. 

## Closing words
Now you know in detail how the ALUP time synchronization works and how it was deducted.

In the future, we can use this synchronized time to coordinate the RGB-Colors of multiple Receivers so they can update their LEDs at the same time. Furthermore we can minimize latency and jitter using time stamps in every ALUP Frame.

For reference implementations, see [Python-ALUP](https://github.com/Skyfighter64/Python-ALUP) (Sender) and [Arduino-Alup](https://github.com/Skyfighter64/Arduino-ALUP) (Receiver). The ALUP Protocol Documentation can be found [here](https://github.com/Skyfighter64/ALUP).
<!--
 See if you can find $t_2$ and $t_3$ somewhere in the protocol headers.
:>
-->