# Power Management and Radio Explorations on the NRF52840


## The NRF52840

Exploring the NRF52840 Dongle plus Power Management.
The nRF52840 Dongle is a small, low-cost USB dongle for Bluetooth® Low
Energy (LE), Bluetooth mesh, Thread, Zigbee, 802.15.4, ANT and 2.4 GHz
proprietary applications using the nRF52840 SoC.

### Notable Features
- Uses the Bluetooth 5 specification, improving long range support 
up to x4 and doubling on-air data rate to 2Mbps
- Ideal for in home application with both BL and [802.15.4](https://en.wikipedia.org/wiki/IEEE_802.15.4)
which is a low rate personal area network. 
- Features an on-chip cryptography hardware accelerator, enables us to be more 
secure and faster with a lower power budget. Which should make encrypted operations 
such as network and firmware verification faster and cheaper.
- Supports  OTA-DFU for field updates
- Core M4 running up t0 64 MHz with a FPU and DSP extensions
- Support 8 and 16 bit SIMD instructions 
- XIP is possible with a wait state that can be optimized by caching.
- Secure Boot support further enhanced by the crypto accelerator aforementioned 
- Flexible power management, up to 0.4&micro;A in System Off
- USB full speed 
- QSPI 
- Type 2 NFC with Wake-on field
- 1MB flash 
- 256kB RAM
- EasyDMA
- i2S Audio support



### Power States & Wake Capacity
Comes with a Power Management Unit designed to ensure maximum power efficacy. 
 
The PMU automatically detects which components need power and clock at any time, 
using this information it will stop and start components to save power. 

* The PMU makes it hard to measure consumption, since it is always tuning the device. 
- Normal Voltage mode if supply to VDD and VDDH exists, with only VDDH it enters high voltage mode.
- On  chip regulators
- Analog and digital wake up pins
- ARM Power States
- Up to 0.4&micro;A in system OFF 
- Up to 1.5&micro;A in system ON
- 15 level low-power comparator with wake-up from system off 
- NFC with wake-on field
- RTC Wake-on
- Wake in Debug Interface mode

Most of this can be controlled from the RESETREAS register.

#### Wake Up Sources

- WIC wake up interrupt controller. 
- Analog or digital wake up pin in the power supply system.
- LPCOMP, low power comparator can be used as a source from 
System OFF mode, this also functions as an analog wake up source.
- NFCDT near field communication field detect can also be used as a wake up source. 

### Radio System & BLE
The 2.4GHz Radio rx is compatible with multiple standards such as 1Mbps, 2Mbps and
Long Range BLE, including IEEC 802.15.4 and Nordic's proprietary 1 & 2 Mbps modes. 

- Multi domain transceiver. 
- Low link budget plus low power operation
- EasyDMA support makes it efficient at data interfacing say from `PDM -> RADIO -> External`
- Automatic address filtering and pattern matching, this makes it easy to whitelist addresses
and simplifies inter frame spacing. i.e it sort of has a known list of addresses and takes care of 
timing to the CPU has less work. 
- Included is a bit counter that can signal ow many bits have tx or rx. 
- Also include a received signal strength indicator. 


Radio modes can be chosen using the Preamble in the Radio Packet. 
1. BLE 1Mb one byte 
2. BLE 2Mb 2 bytes 
3. BLE LR125, LR500  10 repetitions of 0x3C
4. IEEE_802_15_4 it is 4 bytes long all 0s


### PPI/DPPI 
The programmable Peripheral Interconnect allow peripherals to interact with each other 
independent of the CPU... DMA? It seems a little different since this is not just moving data. 

One can program the behavior expected to PPI. Is this a co processor?

It provides a mechanism to trigger a task in one peripheral as a result of an event occurring in another 
peripheral, through a PPI channel.

A channel consists of three end point registers one EEP (Event End Point) and two TEP(Task End Points)
A task is connected to a TEP via the task register address and an event is connected to an EEP via the 
event register. 

Think of this as a direct link between hardware blocks in the MCU. Let's make a small example if 
you want to trigger an LED every time a timer reaches a certain value you can 

Timer Interrupt -> CPU wakes up  -> Toggles LED -> CPU goes to sleep 

with PPI we can 

Timer Reaches an (EEP) -> PPI Channel -> Triggers LED (TEP)


The objective today was to explore the NRF52840  and get familiar with its peripherals mostly RADIO 
in addition to this since we are exploring EDGE applications we need to be keenly aware of how power 
is used and saved in such a system. 


