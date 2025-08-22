# Power Management in Zephyr

Previously on a high level we explored the [NRF52840 and its Radio and Power capabilities](https://gaiaochos.com/2025/08/21/NRF52840-Power-management-and-Radio.html).
This time let's take a small break from theory and perhaps explore a few practical implemementations, 
that way we can build an intuition for real world power management and the options we have. 


## PM_DEVICE and PM_STATE
Zephyr has a power management subsystem implementation designed to be arch and SOC independent, achieved
by separating the core PM  from the SOC specific components of the implementation. These features are 
naturally classified into System and Device power management categories.
Additionally the system allows components to block the system from transitioning states effectively allowing 
fine grained control to prevent loss of context see [here](https://docs.zephyrproject.org/latest/doxygen/html/group__subsys__pm__sys__policy.html#gabbb379f8572f164addafe93ad3468f3d)

#### System PM
System power management is enabled by [CONFIG_PM](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_PM)
this option enables the kernel to call the PM system when entering idle. 
With System Power Management it is the apps responsibility to set a wake event, the app also works with the 
PM system to define how long it expects to sleep and what sleep level to transition to.
Note that the `POWER STATE` may dictate what peripherals can be used as wake sources. 
Combined with `PM_DEVICE` this can offer even more fine grained power savings. 

The PM subsystem defines a set of `pm_state` in general lower power states (higher index in the enum), will 
offer greater power savings but also `have higher wake latencies`.

##### Policies 
The PM system has a policy manager that decides which state to transition to depending on the policy. 

- It can only choose between stets defined for the platfom.
- Other factors such as locks and disallowed states also affect this as well do min and max latencies.

The following Policies are Supported

1. Residency Based: 
System will enter the state the offers the most savings, the constraint being the sum of 
the minimum residency value and the latecy to exit the mode must be less than the scheduled idle time.
see the `zephyr,power-state` link.

```c
if (time_to_next_scheduled_event >= (state.min_residency_us + state.exit_latency)) {
    return state
}
```

2. Application Based:
In this policy the appliction implements the `pm_policy_next_state()`, which enables the application to decide which power state thesystem show transition to based on the remaining time until the next scheduled timeout. 
An example of [this policy](https://github.com/zephyrproject-rtos/zephyr/blob/main/tests/subsys/pm/power_mgmt/)

#### Device PM
Basically this is a mechanism by which of the power management actions of device drivers can be affected in unison 
based on expetations that can be set by any component of the system. 


##### Device Runtime PM 
Coordintated interaction between drivers, subsystems and applications. Higher levels with better context are allowed 
to control lower levels. Each layer can operate independently since the PM subsystem uses reference count to suspend 
or resume. 

- Device Drivers: Manage device power states, by way of the [device runtime PM API](https://docs.zephyrproject.org/latest/services/pm/api/index.html#device-runtime-apis)

- Subsystem such as network, video  may have more context which allows then to make informed decisions about when to
suspend or resume. 

- Applications: The can impact device power management as well as they have a lot of context on when they need what. 

In this state the system can change power states quicklu because it does not need to spend time on devices that are 
runtime enabled. 


##### System-Managed Device PM
In this method devices are suspended along with the system enetering a SOC power state, it uses 
`CONFIG_PM_DEVICE_SYSTEM_MANAGED`. 

Here the PM subsystem will check whether the selected state trigers device power management, and if it does 
it will susped the devices and on wake up resume them in reverse order so deps are taken care of. 

Note: This system has drawbacks and Device RUNTIME PM is the preffered method. 


EX: Write small code snippets for `pm_device_action_run()` and system state transitions

## Logging Current Consumption
- We can use an ammeter if the our development kit allows like the STM32F4 Discovery 
- We can use Shell Commands to check if indeed a device has transitioned in it power mode. 
- There is a PM_STATS config

- Research tools: Power Profiler Kit II, Otii Arc, shunt + DMM

##m Wrap-up
[PM_DEVICE](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_PM_DEVICE)
- This enables the device power management interface, enabling drivers to perform in detail operatoions 
like turning off clocks and devices.

[PM_STATE](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_POWER_DOMAIN_SOC_PM_STATE)
- Generic power domain across SOCs enables transition in and out of states.
- more information on states can be found in the [zephyr,power-state](https://docs.zephyrproject.org/latest/build/dts/api/bindings/power/zephyr%2Cpower-state.html#std-dtcompatible-zephyr-power-state)
- Differences between `pm_device` and `pm_state`
