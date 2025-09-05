# Event Driven Architecture for BLE, Temp and Mic App with FSM

## **Review Current App and Identify Changes**
Explore where FSM will fit and which threads/queues can be removed.

- List all threads: Temp thread, BLE thread, mic thread.  
- List all message queues: temp queue, others.  
- Identify current global variables used for BLE or sensor data.  
- Decide which data belongs in FSM context (`last_temperature`, `mic_detected`).  

---

## **Design FSM Context and Events**

Recap on FSM Events and Designs from previous sessions.

---

## **Refactor Threads to Post Events**
Make threads event producers only; remove BLE thread.

- Temp thread: post `EVENT_TEMP_READY` with reading to FSM event queue.  
- BLE thread: remove entirely — BLE read handler will query FSM context directly.  
- Mic thread (if any): post `EVENT_MIC_DETECTED` events.  
- Ensure no threads are directly modifying global variables that now live in FSM context.  

---

## Update BLE Read Handler**
Make BLE read handler pull data from FSM context.

- Replace reads from old globals or queues with reads from FSM context (or via getter API).  
- Ensure read handler is safe and non-blocking.  
- Verify BLE characteristic returns latest `last_temperature` (and mic status if needed).  

---

## **Verification and Cleanup**
Make sure FSM integration works as intended.

- Start the app, connect BLE, verify temp readings are available on demand.  
- Ensure FSM states transition correctly when BLE connects/disconnects.  
- Remove old unused globals, BLE thread, and temp queue if fully replaced by FSM.  
- Optional: log FSM state transitions for debugging.  

---

## **Review & Outcome:**  
We now have an **event-driven application via FSM**, with:
- Threads as event producers  
- FSM as single source of truth for system state and sensor data  
- BLE read handler directly pulling from FSM context  
- Clean, maintainable architecture ready for future extensions.

