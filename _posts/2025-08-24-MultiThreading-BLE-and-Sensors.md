# RTOS Threads, Queues Async BLE & Sensors

## (15 min) – RTOS Threading in Zephyr.

#### k_thread
Zephyr offers threads which are Kernel Objects that are used for 
application processing that is too lenghty to do in ISR. 
We should note that for appropriate Scenarios we can use Zephyr without threads 
by setting  `CONFIG_MULTITHREADING=n`.

- We can have any number of threads limited only by RAM 
- Each thread jas
    1. A stack for memory, can be tailored to the thread 
    2. A controdl block for metadata 
    3. An entry point function 
    4. A priority to determine resource dedication. 
    5. A set of options for kernel decisions.
    6. A start delay 
    7. An execution mode priviledged or unpriviledged. 
- A threads lifecycle 
    Thread Creation -> Executes(Usually forever) | Can be aborted -> Sometimes termination

##### Thread States 
- Ready to be executed 
- Unready for execution due to 
    1. Hasnt been started 
    2. Waiting for a kernel object (semaphore, mutex or such)
    3. Waiting for a timeout
    4. Has been suspended 
    5. Has been terminated or aborted 


##### Thread Priorities 
This is an integer value that can be negative or non negative, Numerically lower 
values indicate higher priorities. Thread A at -2 is more important than thread B at 5.
With priorities we have  a number of classes,  
    - Cooperative threads, this has a negative thread it stays the current thread until
    something makes it unready. 
    - Preemptible thread has a none negative value, it stays the current thread until it 
    it is supplanted by some other thread of higher or equal priority becoming ready

**Thread priorities can be changed after creation, both up and down** which also means
threads can change between the above classes. 

Perhaps this can be a way of implemeting `critical sections`. 

- Threads have a 32 bit custom data aread available onl;y for themselves to use as they choose.

##### Spawning 
We spawn a thread by defining it's stack area, control block and then calling 
`k_thread_create`

#### `k_work` – asynchronous work queues
This is a kernel object that uses a dedicated thread to process work items in 
a FIFO manner. 
Typically we use work queues to offload non-urget processing to a lower-priority thread, 
this way we preserver time-sensitive processing. 

A work queue will have 
    a) A list of work items that have been added but not processed 
    b) A thread that processes the work items in the queue 
The work queue will always yield between submitted work items, that way we avoid starvation. 
It must be initialized before it is used.

##### Item Lifecycle
- Any number of work items can be defined and they are referenced by their address. 
- An item is assigned a handler function that accepts a single argument, this function 
is the one used to process the work item, basically a queu can have varied items with 
different processing. 
- It must be initialized before it can be used, here a handler is chosen and the item is 
marked as not pending. 
- It can be queued by submitting to a workqueue by an ISR or a thread, this appends it to 
the existing queue. 

Depending on the scheduling priority of the workqueue's thread an item could be processed quickly 
or over an extended period.

- Work items have status and sometimes can even be in multiple states 
    a) running on queue 
    b) queued to run on the same queue again 
    c) marked cancelling
    d) scheduled to be submitted to a queue
or even all of them, in any of these states its consideed `k_work_is_pending()` or `k_work_busy_get()`

##### **Handler Functions**

These can use any kernel API available to threads, blocking operations should then be handled with care. 
Though they take an argument it can be ignored. 
A handler function can resubmit its work item. Remember FIFO by it being in the handler function 
it has been dequeued. This allows the handler to execute the work in stages
- Consider a networking scenario where information had to be got, formatted then maybe pushed, such a feature allows the processing of other items in the queue without delay. 


#### Delayable Work 
Threads or ISRs may need to  schedule items, rather than make them ready, immediately.
This a normal workitem however with an added field that records when and where thr item 
should be submitted. 

Built as a normal item with different Kernel APIs that case it to be latched to a timeout 
and only submitted after.


#### Triggered Work 
This is also a standard work item that has the following properties added. 
- A pointer to poll events that may trigger work submissions
- A size of the array containng the poll events

These enable us to schedule work in response to a poll event , where a user defined 
fuction is called when a resource becomes available ot a poll signal is raised, or a timeout occurs.

again here submission is the same as the normal work item but with different kernel APIs 
_these can be cancelled depending on whether they are still waiting or not_

## System Workqueue 
The Kernel defines a workqueue known as the system workqueu, which is available to any 
application that needs a workqueue,it is optional and only exists if the applioication uses it. 


We can define a workqueue in Zephyr as follows 

```
#define MY_STACK_SIZE 512
#define MY_PRIORITY 5

K_THREAD_STACK_DEFINE(my_stack_area, MY_STACK_SIZE);

struct k_work_q my_work_q;

k_work_queue_init(&my_work_q);

k_work_queue_start(&my_work_q, my_stack_area,
                   K_THREAD_STACK_SIZEOF(my_stack_area), MY_PRIORITY,
                   NULL);
```

- Understand **thread priorities** and **preemption** in Zephyr.
 Which primitive to use for sensor → BLE communication?

---

## Design the Pipeline
- Sketch a **thread architecture**:
  - **Sensor thread**: reads temperature every 
  Which module will own the sensor thread, in our case the main will own all
  Place definitions in common header for reuse across the app.
  - **Queue**: `k_msgq` to hold temperature messages
  a) How big of a queue 
  b) How will we invalidate
  c) Do we latch anything to the work queue
  - **BLE thread**: consumes queue, updates characteristic,
  a) also owned by main
  b) Should be the consumer
  c) Should this also have its own queue
  - `k_mutex` / `k_sem` – synchronization primitives
  That we didnt need to use in this version perhaps with notification
- Decide on **message structure** (`struct temp_msg { float value; }`).
- Define **queue size** (e.g., 2 messages). so we can explore invalidation
- Identify points where **workqueues (`k_work`)** are needed for ISR-safe BLE notifications?


---

## Implement Sensor Thread
- Create a Zephyr thread for sensor polling:
  - Fetch sensor using `sensor_sample_fetch()` and `sensor_channel_get()`.
  - Convert to `float`.
  - Push to `k_msgq` using `k_msgq_put()`.
- Handle queue full case (drop oldest or overwrite).
- Test thread in isolation: print values using `LOG_INF` to ensure correct readings.

---

## Implement BLE Thread
- Create BLE thread consmer thread:
  - Pull from `k_msgq` using `k_msgq_get()` non-blocking.
  - Update characteristic value (`float temp_value`).
- Ensure BLE reads (`read_custom_characteristic`) can safely access latest value.
By updating the value everytime the thread fetches, and invalidation in the sensor thread
- Test BLE characteristic with nRF Connect / BLE Scanner: values should appear as floats.
`Values` are still hex, so raw binary.

---

## Integrate and Test
- Start sensor thread and BLE thread from `main()`.
- Verify:
  - Sensor values appear on the BLE app
  - Queue correctly passes data
  - Threads run concurrently without blocking
- Optional: adjust thread priorities and queue size for responsiveness.

---

### Outcome
- Modular Zephyr project with **sensor → queue → BLE** architecture.
- Threads fully decoupled; BLE characteristic shows live temperature.
- Ready to extend for multiple sensors or notifications.

