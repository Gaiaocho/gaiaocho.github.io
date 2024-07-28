# To Stack or To heap FreeRTOS Allocation 

A runbook on how to deal with Heap memory allocation with FreeRTOS. FreeRTOS 
checks `config_SUPPORT_DYNAMIC_ALLOCATION`. If this value is undefined or set to
the value of 1, FreeRTOS needs a heap allocation scheme to be included. Usually 
one would choose between `heap_1.c` through `heap_5.c` additionally we can provide  
our own [heap allocation implementation](https://www.freertos.org/Static_Vs_Dynamic_Memory_Allocation.html).


## Basics

### Types of Program Memory 

#### Stack Memory
- Used for static allocations, variables and function call management.
- Managed (allocated and deallocated) by the compiler.
- Typically smaller than heap in size of structures.
- Fast Access
- Predictable

Static allocation is more deterministic, reduces risk of memory fragmentation 
and allocation failure risks as well.

In FreeRTOS the API using static allocation can be enabled by 
`config_SUPPORT_STATIC_ALLOCATION`.


#### Heap Memory
- Used for dynamic allocation, as in on the fly.
- Managed by the programmer.
- Size is limited only by what system has available.
- Objects can live as long as they need.
- Suitable for large structures e.g queues and lists.
- Scope is not limited.


Heap memory is helpful in scenarios such as when incoming data size is unknown,
working with flexible data structures, Dynamic resources like files and sockets.
It isn't just simple global variables as it can be resized and manges during a 
program's lifetime. 

Dynamic allocation also simplifies design and planning. It minimizes the 
RAM footprint.

It is valid have both allocation schemes set and in use at the same time.


#### Malloc & Free

[malloc](https://en.cppreference.com/w/c/memory/malloc) 
- Allocates bytes of uninitialized storage.

[free](https://en.cppreference.com/w/c/memory/free)
- Deallocates storage previously allocated by malloc()

- These are the functions in the standard library for memory management. Sometimes 
these functions aren't available in the run time of embedded systems.
- Their implementations can be huge affecting system specifications
- They aren't deterministic. 
- They can cause memory fragmentation. 
- They can complicate linker set up.
- The can cause difficult bugs to debug. 


## Heap Allocation Schemes in FreeRTOS.

Due to the complications listed above. FreeRTOS treats memory allocation as part 
of the portable layer. Different systems will need different allocation and timing 
requirements. Hence as explained earlier  FreeRTOS provides a number of implementations
plus the ability to provide our own.

FreeRTOS uses pvPortMalloc() and vPortFree() in place of malloc() and free().



### Heap 1 
For applications that create kernel objects before starting the FreeRTOS scheduler.
In this scenario it is expected that dynamic allocations are done before 
starting the scheduler and will stay as such until the application stops.

- This negates the need to consider determinism and fragmentation.
- Does not implement `vPortFre()`.
- heap_1.c is deterministic and can be used in cases where dynamic memory allocation
is prohibited.


### Heap 2 
Superseded by heap 4 and not recommended for using in new designs we will not be 
discussing this scheme much.

- It does implement `vPortFree`
- It is fast
- Not deterministic.


### Heap 3
This scheme uses the standard `malloc` and `free` and requires the linker to 
define heap size.

`config_TOTAL_HEAP_SIZE` is not used in this scheme.

This scheme makes `malloc` and `free` thread safe by suspending the scheduler 
during there invocations.

### Heap 4
This scheme works by subdividing memory  into small blocks each time an allocation 
occurs.

`config_TOTAL_HEAP_SIZE` is used. 

- Uses a first fit algoritm, i.e use the first fitting block of memory. 
- Combines adjacent free blocks, thereby reducing risks of memory fragmentation.
- Suitable for applications that repeatedly allocate different sized blocks of RAM.
- It is not deterministic but it is faster than standard `malloc` and `free`.


### Heap 5
This uses the same as scheme as  Heap 4. The advantage of this scheme is that
it can combine memory from multiple separated spaces into a single heap.

- Useful when  the system RAM does not appear as a single contiguous block in 
systems memory map.
- Some work is needed during initialization to inform FreeRTOS of the various 
RAM regions. 



## Heap Utility Functions & Macros

Sometimes we need to place the heap at a specific location, for this we can use 
`config_APPLICATION_ALLOCATED_HEAP`, this configuration allows us to specify the 
start of heap in our application.
When this it is enabled or undefined the application must allocate an array 
called `ucHeap` whose dimensions are `config_TOTAL_HEAP_SIZE` refer to your 
compiler for syntax required for this. 

In `GCC` it would be 

```c
uint8_t ucHeap[config_TOTAL_HEAP_SIZE] __attribute__ ((section("myheap"));
```


- xPortGetFreeHeapSize: number of free bytes in the heap at the time the function
is called. 

- xPortGetMinimumEverFreeHeapSize: minimum number of unallocated bytes since 
application started executing. This can be used to monitor how close our
application is to running out of memory o to see how much we are using.

- vPortGetHeapStats: Returns a structure with heap usage that shows, available bytes,
largest free block, smallest free block, number of free blocks, minimum bytes ever 
remaining, successful allocations and frees.

- vTaskGetInfo: Gets information on a specific task's memory usage.

- Malloc Failed Hook: We can provide a callback for when allocation fails this 
needs a configuration setting to work. Allows us to implement graceful recoveries 
to memory allocation failures.



### Using Fast Memory
We can use `pvPortMallocStack` and `vPortFreeStack` to optionally enable a scheme 
of our own.



## Static Memory Allocation 
Dynamic allocation comes with some disadvantages and some caveats in order to 
avoid these we can use static allocation.

Here
- All required memory us known at compile time.
- All memory is deterministic.

We enable this by setting `config_SUPPORT_STATIC_ALLOCATION` this causes the 
kernel to enable static versions of the kernel functions. 

When static memory is enabled and timer and idle task will be use static memory 
provided by the user implemented functions.


Find more in depth information on this on the [FreeRTOS Site](https://www.freertos.org/index.html)

Thank you for checking out these notes!





























