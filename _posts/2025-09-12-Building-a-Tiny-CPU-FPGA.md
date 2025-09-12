#  A tiny 8 bit CPU on FPGA

Previously we went through a [Verilog Teardown](2025-09-11-FPGA-Horizons-with-Upduino3.md) where 
we inspected and walked through the sample code that came with our [Upduino Devboard](https://tinyvision.ai/products/fpga-development-board-upduino-v3-1). 
As we went through this I was quite clear that my intention here was to use the FPGA to 
learn digital design and my intended path to that was by implementing a RISC cpu. 

Wel as my initial foray into the excercise I will implement a super tiny RISC style CPU 
while I really wanted to do it from scratch I find composing concepts onto something I understand
will be much easier than doing it from scratch. So that's what we will do today. 

We are going to modify our simple LED driver and rather than just using a delay we will implement 
a small processing unit that goes through a loop and as it does that updates the states of 
the LEDs. i.e the cpu will decide when to light up the LEDS rather than counter states. 

This will be simpler than it seems all we need to add is a 
- CPU Register: To store both data and addresses, in our case a 3 bit value mapping to our LEDs
- A program counter: Or even more obviously an Instruction pointer this points to the function we are running, and is incremented after we fetch an instruction.
- Some ROM memory: Some Read Only memory to store the instructions we will run, usually code goes
but that involves a compiler and loading which makes this complex we want simple so just
instructions for now.
- A slow clock: That is here to give us human visible delays that way we can appreciate what the 
CPU is doing on a human time scale

And a bit of logic that combines all of this.

## Clock Division / Frequency Counter
 The first change we will add is a way to slow down the FPGA clock to a human-visible rate. 
 This way we can observe and appreciate the decisions being made by our cpu.

 ```v
 wire slow_clk = frequency_counter_i[24];
 ```
All this does is it sets the value of the wire when the 25th bit is set and that gives us a slow
clock upon which we will process an instruction.

Remeber this is based on a binary counter and this basically means this will be set 
every 2^25 cyckles.  
- This is simply getting **slow clock signal** from a fast internal clock.  

## Tiny CPU Registers & Program Memory
CPU concept implemented in FPGA.  

-  **registers**:
  - `A` as the main CPU register (3-bit in our case).  
  - `PC` as program counter.  
- **ROM as program memory**:
  - How to initialize with instructions (`initial begin ... end`). This construct
  is used in FPGAs to do some configuration on power up before any clock cyle is run. 
  consider this to be something like placing stuff in RAM so you can use it. 
  We set defaults here, we can load a program or instructions and also assign values to registers.
  - The concept of sequential instruction execution (fetch → execute → increment PC) 
  let's go through it.  

## CPU Instruction Execution
How to implement instructions in Verilog.  

- We first fetch an instruction using the `ROM[PC][7:4]` where we read the higher 4 bits that 
encode our instructions `0001` for Increment, `0010` for output.
- We then use`case` statements for getting our opcodes, i.e what to do.  
- The `INC` instruction with wrap-around uses bit masking to set the current LED to light.  
- Since we had set output to out general purpose register now `OUT` as lets the loop 
run and we get a side effect  

## RGB LED Driver Integration
Map CPU register outputs to RGB LEDs using FPGA primitives.  

- We Studied the `SB_RGBA_DRV` primitive in the previous section and it works as follows
  - `RGBLEDEN` – enable signal  
  - `RGBxPWM` – individual LED PWM inputs  
  - `CURREN` – current enable  
The only difference now is rather than picking the bits we should be sensitive to 
we are now using the value in the Regiser A!

## Putting It All Together
Combine oscillator, clock divider, CPU, and RGB driver.  

- The **slow clock controls CPU execution** for visible LED changes. 
- We run the processor on the rising edge of the slow clock
- We then use the output of the processor in the RGB driver

And Voila! we made a computer! that counts and lights LEDS as it counts!


## References 

[8bit Breadboard FPGA](https://austinmorlan.com/posts/8bit_breadboard_fpga/)

[Processor Register](https://en.wikipedia.org/wiki/Processor_register)

[Program Counter](https://en.wikipedia.org/wiki/Program_counter)
