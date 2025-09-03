# Debugging Usage Faults - Explorations with GDB

## [Fatal Errors in Zephyr](https://docs.zephyrproject.org/latest/kernel/services/other/fatal.html)

- Zephyr provides mechanism for inducing fatal errors
a) We have runtime [assertions](https://github.com/zephyrproject-rtos/zephyr/blob/main/include/zephyr/sys/__assert.h)`CONFIG_ASSERT`
And we can use `assert_post_action()` to handle this sorts of faults. 
b) We can also use `BUILD_ASSERT` for  build time assertion bringing the error closer to development than in production.
c) Applications can also trigger a `k_oops()` which is a fatal error indicating an unrecoverable 
condition in the application logic. 
d) Applications can trigger a `k_panic()`which can not be used by threads in user mode and cannnot 
be handled by the `k_sys_fatal_error_handler`
e) Spurious Interrupts will cause faults if a hardware interrupt that is unconfigured is received.

f) Stack Overflows if a thread pushes more data than it stack provides,  then a fatal error will be 
raised as well. 
In User mode  overflows  are caught and the thread would not be allowed to write to
adjacent memory, Supervisor threads may not have the same amenities. 

Some architectures may support `CONFIG_HW_STACK_PROTECTION` and this may catch overflows 
in supervisor threads.

Others may only support `CONFIG_STACK_SENTINEL` which is a software only overflow 
detection feature, it provides no guarantees on memory protection as it is run periodically. 

Finally Zephyr supports `CONFIG_STACK_CANARIES` which are placed at function stack frames and 
are checked at function exit and if overwritten will cause an fatal fault.


**Applications can handle these errors when permitted by utilizing an implementation of
`k_sys_fatal_error_handler()`**


## Grounding in Cortex-M Faults
  - Review **ARMv7-M Architecture Reference Manual**.  
  ### UsageFault 
  This fault handles non-memory related failts caused by instruction execution, causes include. 
  a) undefined instructions, 
  b) Invalid state on execution 
  c) Error on Exception 
  d) Attempting to access a disabled or unavailable co processor
  e) Access to unaligned address 
  f) Division by zero

  This fault can be disabled and if disabled, this fault is escalated to a HardFault

  **EPSR (Execution Program Status Register)**.
  This register contains the T bit used to indicate the processor executes Thumb instructions. 
  Updates to the PC that comply with Thumb instructions must update the T bit, however the T bit 
  must be set i.e `1` if it is reset i.e `0` this causes an invalid state see `b)` above. 

  So when you see `pc=0x00000000` typically occurs (bad pointer execution), it indicates that 
  the aforementioned invalid state occured. 

  - `lr` (link register) : used to store the return link 
  - `pc` (program counter) : holds the address of the current instruction plus 8 bytes
  - `xpsr` These are special purpose program registers that hold statuses


---

## Reproduce and Gather Evidence
- Re-run the firmware with these options enabled in `prj.conf`:
  - `CONFIG_DEBUG=y`
  - `CONFIG_PRINTK=y`
  - `CONFIG_ASSERT=y`
  - `CONFIG_STACK_SENTINEL=y`
  - (optional) `CONFIG_DEBUG_THREAD_INFO=y`

- Observe:
  - Exact point of crash (immediately at startup or after "start streams").
  - Whether it faults consistently at the same address.

  - Value of `lr = 0x080053b1` → map to source with:

    ```bash
    arm-none-eabi-addr2line -e zephyr.elf 0x080053b1
    ```


---

## Map the Registers
- Use **addr2line** or GDB to interpret:
  - `lr = 0x080053b1` (call origin).
  - `r0 = 0x08005fec` (potential function pointer, ISR table entry, or buffer).
- Form hypotheses:
  - Null function pointer call?
  - Overwritten stack returning to `0x00000000`?
  - Misconfigured interrupt vector table?


---

## Deep Dive in Zephyr Context
- Inspect the **codec sample** you are running.
- Locate the point where "start streams" is printed and set a breakpoint there.
- In GDB:
  - Step through execution after this log.
  - Inspect the stack before the `blx`/`bx` instruction that caused the fault.
- Cross-check:
  - Any callbacks registered as NULL?
  - Driver API `.ops` structs properly initialized?
  - DMA/codec driver IRQs correctly configured?

---

## Hypothesis and Fix Path
- Common fixes to test:
  - Initialize all function pointers before use.
  - Check driver API structures (`.ops`) for NULL entries.
  - Verify IRQs are connected with `IRQ_CONNECT`.
  - Validate stack sizes. Enable `CONFIG_INIT_STACKS=y` and use `k_thread_stack_space_get()` to check usage.
- Add defensive assertions:
  ```c
  __ASSERT(func_ptr != NULL, "Function pointer is NULL");
  ```

Dump assembly 

```bash 
arm-none-eabi-objdump -d zephyr.elf | grep -A20 8053b1
```
