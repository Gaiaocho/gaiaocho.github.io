# RTOS the What? and the Why?

```markdown
    Time
    |
    |
    |    Running Task A
    |------------------------------------------------------
    |                    ^     ^     ^
    |                    |     |     |
    |                    |     |     |
    |                    Interrupt Occurs
    |    Task Switch    / \  / \  / \ 
    |-------------------\ /  \ /  \ /----------------------
    |                     \    /    /
    |                      \  /    /
    |                      Interrupt Service Routine (ISR)
    |------------------------------------------------------
    |    Task Switch      / \  / \  / \
    |--------------------\ /  \ /  \ /----------------------
    |                     /    \    \
    |                    /      \    \
    |                    \______|     \
    |                             |    \
    |    Running Task B           |     \    
    |------------------------------------------------------
                      ^          ^
                      |          |
             Interrupt Latency   Task Switch Latency
```


In this little series we are going to explore, Real-Time Operating systems, how 
they work, and when to use them. 

Realtime Computing is all about time predictable computing, i.e computing that
happens within a certain timeline. A common misconception is that realtime computing 
is about real fast computing that occurs as the event in question occurs. 
Here correctness of a computation depends on both its logic correctness and whether
it is done before a certain deadline. A late response is considered a wrong response
in Realtime Embedded Systems.


## Real Time Applications (RTAs)
Real time applications fall into two categories, Hard Real Time and Soft Real Time. 

#### Soft Realtime
Soft Realtime applications, these are applications that have a deadline, however 
missing this deadline isn't catasrophic and would be more of an annoyance. C
Consider a keyboard that misses some keystrokes or jams and blurts several keys 
at once. This would be annoying but not very catastrophic.


#### Hard Realtime
Hard Realtime applications, are applications with a defined deadline, this deadline 
if missed could likely be catastrophic. Consider an Antilock brake system in a
car. If the deadline for when to release the locked brakes are missed the vehicle
could easily slide out. Likely endangering all live on board and in some situations
even least to loss of life where such as system would be expected to do the opposite



## Real Time Operating Systems (RTOS) 
Real time applications need a **Real Time Operating System** (RTOS), these have
the following qualities:

- Determinism, repeeting an input will produce the same result as before.
- Performant RTOSes are designed to respond very quickly to stimuli.
- Safety and Security, since they are used in mission critical systems RTOSes
are expected to be very safe in their operations.
- Priority based scheduling, this is where there is a stack difference to other 
operating systems. An RTOS will primarily use priorities when selecting which 
task to engage.
- Lightweight, an RTOS should have a small footprint considering the are used 
mainly in microcontrollers or small microprocessors.

RTOS are often used in embedded systems which are systems that are part of larger 
operation as the name `embedded` implies. Often these systems would be considered 
intelligent edge devices i.e they produce and operate on a set of data.

### Tasks in a Real Time World
When learning about real time operating systems and their applications, one will
no doubt come across the terms `Tasks` and `Threads`. They are mostly used interchangeably 
and in this article we will refer to them as `Tasks`. 

A task can be considered as a unit of work to be accomplished by a device or a
process. Consider this essentially to be a small program that runs independently 
performing specific functions.

The stark difference between RTOSes and GPOSes (General purpose operating systems)
is in how they handle task scheduling. 

#### GPOS Task Scheduling 
- They are optimized for throughput, with the goal being to process as many tasks as possible.
- They schedule based on a fairness policy, so no one task will be allowed to 
hold the resources for too long.
- They provide no priority based guarantees.


#### RTOS Task Scheduling 
- Mainly use priority based scheduling though they can be tweaked to cater for 
time slicing and cooperation as well.
- They will only premempt the current task if a task of a higher priority becomes ready.
- They don't focus so much on throughput, albeit they are still performant and fast.
- They operate schedule in a very deterministic style and hence try to be as predictable as possible.
- They provide very low task switching latency, which is the time that passes between a task 
being ready and being switches to.
- They provide low interrupt latency, this is time it takes to switch from an interrupt 
back to a task or on to another task. 
- They offer mechanisms on how to deal with priority inversion which can happen 
quite  often in GPOS systems. 




### Reasons to choose an RTOS as opposed to ISR based or GPOS based
1. Allows you to abstract timing from your application logic.
2. Since you can focus on your application code rather than timing and scheduling
your application can grow and advance as it needs to.
3. They are modular, as they allow your application to run as a set of separate, 
independent tasks.
4. Allows you to develop your team since work can be cut out and worked on independently.
5. Better reuse of resources and code.
6. Better efficiency, with a good RTOS and planning one can end up with a tickless 
system meaning that most idle time can be spent in power saving mode and this 
contributes to efficiency. 
7. Flexible interrupt handling, a good RTOS would allow you to switch up priorities and 
deadlines on the fly thereby allowing one to build a very flexible system. 
8. Mixed processing, we can have both preemptive and cooperative work tasks in our 
system and a good RTOS will aid in this immensely.



These are notes in preparation for some RTOS deepdiving in the DevHeads workgroup if
you feel I have missed anything please do let me know ask questions in any of the channels 
in my contacts below. 



















