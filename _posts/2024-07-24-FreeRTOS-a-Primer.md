# FreeRTOS a Primer


In a [previous post](https://gaiaocho.github.io/2024/07/20/RTOS-the-What-and-the-Why.html) we spoke about RTOSes. 
Where, how and why one would use an RTOS. 

Today I would like to delve into one such RTOS and in particular, I would like to 
explore [FreeRTOS](https://www.freertos.org/index.html). FreeRTOS is a FOSS project 
originally by Richard Barry and now under stewardship of AWS. FreeRTOS is 
free and can be used for your project without requiring you to expose your source. 



## What is FreeRTOS
FreeRTOS is a real-time operating system `Kernel` distributed as a set of C source 
files that serve as modular libraries with complementary functions.

FreeRTOS is ideal for deeply embedded microcontroller or microprocessor applications
it has features that cater for hard and soft real time applications. It can be used
with a barrage of processor architectures and compilers. Around 23 compilers and 43 
processor architectures are supported. 

### FreeRTOS Port
Each supported architecture and compiler combination is known as a FreeRTOS port
hence you will encounter the term `porting`, when you do remember this is simply 
what it means.

As mentioned earlier FreeRTOS is supplied as a set of sources, some of these source 
files are common to all ports with some being specific to just one port.


## Understanding the Sources
Let's explore some key files that are distributed in FreeRTOS, their significance
and how to use them. Familiarity with these elements will help one  with porting 
FreeRTOS or identifying application specific vs library specific files on a project 
they are working on.


#### FreeRTOSConfig.h
This is an application specific file that is used to configure FreeRTOS to your 
application. It contains a set of constants defined ahead of time, here you will 
tune the kernel to your application. This should be in your application sources 
as opposed to the FreeRTOS source directory.

Note don't include this configuration header directly in your code rather include 
`FreeRTOS.h` and let that handle including configurations at the correct time.

It is recommended that you modify a configuration file from a [port demo](https://github.com/FreeRTOS/FreeRTOS/tree/main/FreeRTOS/Demo)
rather than writing one from scratch.


#### Common Sources and what they provide
- `tasks.c`
    - Core task management.
    - Handles priorities, delays and timeouts

- `queue.c`
    - Implements queues and queue communication.
    - Supports blocking and non-blocking tx/rx

- `list.c`
    - Implements doubly linked lists for task management and event lists
    - Used for scheduling and task state maintenance

- `timers.c`
    - Implements software timers allowing notification after periods 

- `event_groups.c`
    - Provides task synchronization with multiple event channels.
    - Provides back-pressure primitives, consider scenarios where a task has to wait 
    for an event or inform another task to wait for it to finish something. 

- `heap_n.c`
    - Provides memory allocation schemes for dynamic memory management. 
    - heap_4.c is popular for its simplicity and best-fit algorithm.

- `port.c`
    - Port layer for the specific architecture that one us using e.g ARM Cortex-M7
    - context switching functionality, interrupt handling, and more low level specific tasks.

These sources collectively provide core functionality of FreeRTOS. With each focusing 
on a specific aspect of the real-time operating system. Making FreeRTOS both modular and 
efficient.


### Data Types in FreeRTOS
When wading through a FreeRTOS based application, one is bound to notice quite 
the number of prefixes, naming, and casing conventions. Understanding these 
conventions will take out the `oddity` of the application making it much more 
approachable.

let's start with 


#### TickType_T
This is could be a 16bit, 32bit or 64bit value depending on the `config_TICK_TYPE_WIDTH_IN_BITS` 
setting. This is generally the native datatype for the architecture. Note that 
16bit is good for 16 and 8 bit MCUs however no reason exists to use 16bit for 
32bit and 64bit systems.


#### BaseType_T 
Generally this is the native datatype for the architecture, which is also the most
efficient data type for the architecture.


### Variable names and their prefixes
- `c` for `Char` variables    `cLetter`
- `s` for 16 bit variables    `sShortNum`
- `l` for 32 bit variables    `lLongNum`
- `x` for BaseType variables  `xEventGroup`
- `u` for unsigned variables  `usShortNum`
- `p` for pointer variables   `pParameters`
- `v` for void                `vTaskCreate`



### Functions and their prefixes
Function names in FreeRTOS are generally prefixed with their return type according 
to the above list plus the file where they are created, let's look at some examples
to understand this.

- `vTaskPrioritySet`
    - returns a void 
    - defined in Task
    - name is PrioritySet


- `xQueueReceive`
    - returns the base type 
    - Defined in queue
    - Recieve is the name

- `pvTimerGetTimerID`
    - returns a void pointer 
    - defined in timer.c 
    - GetTimerID is the name

Privately scoped function are prefixed with the `prv` prefix denoting that they 
are local to the file they are defined in.


### Macro Names 
Macro names are prefixed with lower case strings that show where the macro is
defined, with the name of the macro being in upper case. Again some examples.

- `portMAX_DELAY`
    - Defined in port.c
    - MAX_DELAY is the name 

- `taskENTER_CRITICAL`
    - defined in tasks.c 

- `pdTRUE`
    - defined in projdefs.c 

- `config_USE_PREEMPTION`
    - defined in FreeRTOSCONFIG.h

- `errQUEUE_FULL`
    - defined in `projdefs.c`




## Where Next?
With this information one should be able to examine a FreeRTOS based application 
and identify which files are application specific as well as library ones. Additionally
when looking through the source one can use the schemes described above to understand 
the particular elements, where they are from and what they return.


## Porting FreeRTOS
To port FreeRTOS to your specific processor architecture and compiler combination
one needs to do the following.

1. Get your project up and running. Compiling and building without errors.
2. Download FreeRTOS from (download)[https://www.freertos.org/a00104.html]
3. Add FreeRTOS sources to the application and update your compiler as necessary.
4. Copy a FreeRTOS configuration from an applicable demo link above.
5. Remember you may need to remove sysmem so as to allow FreeRTOS memory management as well you may encounter 
issues with some repeated interrupt handlers.
6. Choose and add a heap management option.
7. Include the sources and headers in your build and it should build without errors or warnings 


With this you should be able to jump in a FreeRTOS project and get going!

Thank you for reading see you next time!

