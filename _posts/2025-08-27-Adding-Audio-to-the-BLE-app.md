# Adding Audio capabilities to our BLE application. 


Let's add a mic to our BLE application and receive audio frames into a buffer. That way 
later we can apply some fun signal processing to the audio. 

---

##  What is I2S and how do we use it in zephyr. 
Inter-Intergrated Circuit Sound [I2S](https://en.wi:wkipedia.org/wiki/I%C2%B2S) is a 
serial interface protocal for transmitting two channel digital audio as
Pulse Code Modulation [PCM](https://en.wikipedia.org/wiki/Pulse-code_modulation) betwen 
IC components of an electronic device. 

A bus separates clock and serial data signals which means the receivers are simpler. Since 
the clock does not need to bre recovered from the data stream. 

### DMA Segue
DMA (Direct Memory Access) lets data move directly between devices and memory without 
the CPU handling each piece. It’s useful for streams like audio because it allows 
continuous data capture or playback efficiently, freeing the CPU to work on other tasks 
instead of manually moving every sample. 
 
   - Explore `k_mem_slab` and why this would be  key.

   #### Memory Slabs
   - In zephyr a memory slab is a fixed memory allocator, Imagine a stack of blocks that 
   one can check out use and return.
   - Blocks are all the same size since it is predefined 
   - Allocated at compile time so we dont run a risk of running over 
   - They are thread safe and this guards us from leaking.
   - Since blocks are all the same size it prevents us from memory fragmentation. 

   We just define these blocks and `i2s_read` basically gets a block filled by DMA 
   and if we are ever transmitting the `i2s_write` function will pass a block to DMA 
   for transmission.
   Typically we have 2 Blocks so we can do double buffering i.e fill this one while we 
   process the other one.



#### Pins Used 
- Serial Clock: SCK also known as BCLK 
- Word Select: WS also known as LRCLK or FS 
- Serial Data: SD also known as SDATA, SDIN, SDOUT, DACDATA, ADCDATA

##### Zephyr I²S API
- [Zephyr I²S](https://docs.zephyrproject.org/latest/hardware/peripherals/audio/i2s.html)
- Focus on key APIs such as `i2s_configure()`, `i2s_trigger()`, and `i2s_read()`.
- Understand the `i2s_config` struct and how it defines word size, channels, format, and buffers.

##### Check nRF52840 I²S capabilities
- Verify that `CONFIG_I2S_NRFX` is available in Kconfig.
- Check the board's `.dts` file for an `i2s0` node.
- If missing, plan to overlay the device tree for your specific pins.

   **Audio flow plan**
   - Configure I²S for the mic.
   - Allocate RX buffers.
   - Start reception.
   - Read audio frames into buffers for verification.

---

## Device Tree + Configuration

1. **Device Tree overlay (20 min)**
   - Create a board overlay to define I²S pins: SCK, LRCK (WS), SDIN (data).
   - Use the nrf52840 Reference Manual 
   - Use the [Sipeed I2S_Mic Datasheet](https://dl.sipeed.com/MAIX/HDK/Sipeed-I2S_MIC/)
   - Reference the nRF52840 [pin control](https://github.com/zephyrproject-rtos/zephyr/tree/main/boards/nordic/nrf52840dongle)

2. **Project configuration (10 min)**
   - Enable I²S in `prj.conf` using `CONFIG_I2S=y`
   - Adjust stack and heap sizes to accommodate audio buffers.

---

## Application Flow 

1. **Initialization**
   - Obtain the I²S device handle using Zephyr device API.
   - Configure it for mono 24-bit I²S audio with an appropriate sample rate (16 kHz or 48 kHz).
 
   ``` c
   #define I2S_RXNODE DT_NODELABEL(some_i2s_rx)

   #define SAMPLE_BIT_WIDTH 24
   #define SAMPLE_FREQUENCY 16000
   #define CHANNEL_NUMBER 1
   #define BLOCK_SIZE 256 
   #define BLOCK_COUNT 2
   #define TIMEOUT_MS 
   
   K_MEM_SLAB_DEFINE_STATIC(mem_slab, BLOCK_SIZE, BLOCK_COUNT, 4);
   ```


2. **Start capture (10 min)**
   - Trigger the I²S peripheral to start receiving audio frames.
   - Queue buffers to hold incoming audio, use the mem slab mentioned earlier
   - Read frames into memory buffers.
   - Verify sample values for activity (non-zero, fluctuating with sound).
   - Use UART or logging to monitor audio frames.
   - Ensure the mic outputs valid audio data and buffers are being correctly consumed.

   ```c
   ...
   i2c_configure(i2s_dev, I2S_DIR_RX, &config); 

   i2s_trigger(i2s_dev, I2S_DIR_RX, I2S_TRIGGER_START); 
   /* Capture Frames*/
   i2s_trigger(i2s_dev, I2S_DIR_RX, I2S_TRIGGER_STOP);
   ```

---

## Validation & Debug 

1. **Sanity checks**
   - Confirm pins are wired correctly (SCK, WS, SDIN).
   - Check I²S word size and alignment.
   - Ensure buffers are being filled as expected.

---

## What have we managed to do

- Have the MSM261S4030H0 I²S microphone wired and configured in the device tree.
- Initialize and configure the I²S peripheral in Zephyr.
- Capture audio frames into memory buffers.
- Validate that the microphone is producing correct audio data.

---

**References:**
- [Zephyr I²S API documentation](https://docs.zephyrproject.org/latest/hardware/peripherals/audio/i2s.html)
- [nRF52840 I²S peripheral documentation](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.0.pdf)
- [Zephyr Device Tree Guide](https://docs.zephyrproject.org/latest/guides/dts/index.html)
- [MSM261S4030H0 Datasheet](https://www.memsic.com/products/mems-microphones/)

