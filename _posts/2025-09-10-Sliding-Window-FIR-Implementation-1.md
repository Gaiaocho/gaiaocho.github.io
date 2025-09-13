# FIR Filters for Clap Detection

## Core Concepts  

- **Time vs Frequency**  
  - Audio samples from your microphone are in the time domain.  
  - FIR coefficients are designed in the frequency domain.  
  - Key idea: convolution in time equals multiplication in frequency.  

- **Impulse Response**  
  - A FIR filter is a weighted sum of past samples.  
  - Changing the weights (coefficients) shapes which frequencies pass or get blocked.  

- **Our Project**  
  - Keep 2000–5000 Hz (claps), remove the rest.  
  - This is a band-pass filter specification.  
  - The coefficients you generate will enforce that specification.  

[Convolution to Multiplication](https://en.wikipedia.org/wiki/Convolution_theorem)

Basically even though our source signal is in the time domain, and we designed our filter in 
the frequency domain since we end up doing a pointwise multiplication in the time domain it is
equal to convolution in the frequency domain.


## Apply to Your Codebase  
Integrate FIR into your MCU clap-detection loop.  

- **Set Up CMSIS FIR**  
  - Initialize FIR with your 50 coefficients and state buffer.  
  - Match the block size to your DMA buffer. 

  ```c
  
    #define NUM_TAPS   50
    #define BLOCK_SIZE 4410

    static const float32_t firCoeffs[NUM_TAPS] = {
        // Paste your 50 coefficients here
    };

    static float32_t firState[NUM_TAPS + BLOCK_SIZE - 1];
    arm_fir_instance_f32 FIR;

    void FIR_Init(void) {
        arm_fir_init_f32(&FIR, NUM_TAPS, (float32_t *)firCoeffs, firState, BLOCK_SIZE);
    }

  ```

- **Process a Block **  
  - On DMA callback:  
    - Convert raw PCM samples to normalized floats.  
    - Pass them through the FIR filter. 

  ```c
      void FIR_Process(const float32_t *input, float32_t *output) {
        arm_fir_f32(&FIR, input, output, BLOCK_SIZE);
      }
  ```


- **Detect Claps **  
  - Compute the energy of the filtered block.  
  - Compare to a threshold to trigger clap detection.  
  - Add a short refractory period (about 200 ms) to avoid multiple detections per clap.

  ```c
      float32_t energy;
      arm_power_f32(outputBlock, BLOCK_SIZE, &energy);
  ```


---

## Final Outcome  
- Clear understanding of FIR filters: time-domain implementation, frequency-domain design.  
- Ability to design and visualize your own filter coefficients.  
- Working CMSIS FIR integrated into your project for clap detection.  





