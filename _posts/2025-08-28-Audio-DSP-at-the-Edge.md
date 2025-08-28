# Audio DSP at the Edge!


An approach to implement a simple sound/no-sound detection feature using amplitude thresholding.

---

Review our I2S process & How to signal Process 

1. How are we reading audio blocks.
- We capture a block size of 256 
- They are read into our mem slab
- We have the filled blocks shortly after calling `i2s_trigger` to start
- We should likely free our blocks after calling the same trigger to stop

2. The capture format is.
- 24 Bit aligned container sometimes knows as `S24_LE` or `24 bit in 32 bit`
or  `S32_LE`, the LE here is little Endian. 

3. How many samples per block or frame to analyzei
- Our detection window will be 256  samples, each sample will have 3 bytes 
(this fits into our 2 bit  format above) 
- At our sample rate of 44Khz that means a 5.8ms window or 172 times per a second
`time per sample = 1/sample_rate ~~ 1/44Khz`
`window_time = samples/sample_rate ~~ 256/44100` 
for faster detection we can use a smaller window, for richer detection we can use 
a bigger window.

4. Add a thread and  queue to do this in the background/ we can start by doing it in main.

---

## The Detection Algorithm

1. Define the detection metric:
   - We will be using the Root Mean Square (rms) of the absolute value of the signal, what this
   basically means is we will be measuring both the AC and DC components in this system

   a) DC would account for any interference in our signals environment 
   b) AC would account for the actual sound signal itself. 

   This will basically enable us to do a more robust threshold detection.

3. Computing the metric for each buffer:
   - Everytime we have a memory block we will need to calculate the number of samples in 
   the block each sample is  4 bytes (24 bits with a sign), so  `num_bytes/4 = num_samples`
   - Let's use CMSIS-DSP to compute the rms we want from above 
   a) cmsis works with floating point arrays so we need to scale our 24 bit values
   b) We need to normalize our values

   ```c
   #define SCALER 8388608.0f 
   int sample_count = (BLOCK_SIZE / sizeof(int32_t); 
   
   int_32t *samples = (int32_t *)block; 
   float32_t buffer[sample_count];

   for (int i = 0; i < sample_count; i++) {
        //we have sign extended sample above 
        buffer[i]  = (float32_t) samples[i] / SCALER;
   }
   ```
   
   - After this wecan use `arm_math.h` in our code to get the rms 
   `arm_rms_f32(buffer, sample_count, &rms_value)`
4. Plan the comparison:
   - Compare the computed metric against the threshold to determine if sound is present.

---

## Testing Strategy

1. Define test scenarios:
   - Quiet environment to verify that silence is correctly detected.
   - Introduce controlled sounds (clap, voice) to verify detection.
2. Plan how to observe results:
   - Can we Detect Audio and send a BLE notification
   - Can we advance towards identifying the audio? 

---

## Integration and Optimization

1. Plan how to integrate detection into the main loop without disrupting I²S reads.
2. Decide how frequently detection should run (e.g., once per block or at fixed intervals).
3. Identify opportunities to optimize performance:
   - Use simple arithmetic operations.
   - Minimize memory usage by processing in place.
4. Prepare for future enhancements:
   - Optionally, maintain a running average over multiple blocks.
   - Consider adding hysteresis to avoid rapid toggling between sound and silence.

---

**Outcome:**  
We have a structured approach to implement amplitude threshold-based sound detection in our I²S application, including where to integrate it, how to measure amplitude, and how to test and tune it.

