# FPGAs Hmmm, Let's learn by doing.

I recently received the Upduino 3.1 FPGA board and I am now intent on using this 
board to learn about digital design. There are two ways to do this in my view 
I could go do a course on FPGAs try and walk the long way and eventually get 
comfortable enough to start working on designs.

The other way and what I will do, is I will jump into an existing design, learn 
just enoug to understand how that works, then start working on my project which is 
`Designing a minimal RISC cpu on a FPGA`

To that end the Upduino comes with lots of examples and, even a very detailed 
blog on how to buid  a [RISC Cpu](https://pingu98.wordpress.com/2019/04/08/) more on 
that in later sessions for today lets walk the following code and get a grip on what it 
is doing and how. 

### [LED Blink Code](https://github.com/tinyvision-ai-inc/UPduino-v3.0)

The code starts with this. 

```v 
module rgb_blink (
  // outputs
  output wire led_red  , // Red
  output wire led_blue , // Blue
  output wire led_green  // Green
);
```

`output` here means the signal leaves the module 
`wire` here is a type of connection consider this a datatype
`led_green` here is the name of the signal, the signal is sometimes 
considered the driver of the net (here wire is the net)

### Top Module

These are all encapsulated in a module `rgb_blink` this happens to be our `top module`.

- It is a top module as it does not have a parent.
- It outputs led_red, led_green, led_blue connect directly to FPGA Pins

consider this to be the presentation of our `app`.

Modules may contain other modules known as instance modules, more on this shortly. 


### Primitives 

```v
wire        int_osc            ;
reg  [27:0] frequency_counter_i;

SB_HFOSC u_SB_HFOSC (.CLKHFPU(1'b1), .CLKHFEN(1'b1), .CLKHF(int_osc));
```
We know wires from before these are nets that connect primitives. 

However `reg` we haven't encountered this before, it happens to be a register. Registers 
store their value from one assignment to the next and are used to model data storage.

Here we have a register that is 28 bits wide, named `frequency_counter_i`, later we will be 
using this as a counter and incrementing it with every clock cycle. 

The next line looks very cryptic but luckily it is not 

SB_HFOSC        - this is the high frequency oscillator primitive 

u_SB_HFOSC      - this is a name that we assign he oscillator 

.CLKHFPU(1'b1)  - here we are powering on the oscilator

.CLKHFEN(1'b1)  - this enables our oscillator

.CLKHF(int_osc) - connects the oscillators clock output to `int_osc wire`

In short this says, we have an oscillator of type `SB_HFOSC` name it `u_SB_HFOSC` 
power it up, enable it and connect it's output to `int_osc`. 


## First Bit of Magic!
So far we ahve encoutered lots of declarations and instantiations, now lets go into 
combining these primitives into something that works someway and that brings us to this 
section of code

```v
 always @(posedge int_osc) begin
    frequency_counter_i <= frequency_counter_i + 1'b1;
  end
```

Here we have a few things we sort of know and a few we not encountered, this is a loop. 
`always`, `posedge`, `begin` and `end` are reserved keywords and instantly that makes this code 
 alot simpler. This means whenever the signal at `int_osc` is moving from 0 to 1 `posedge`
 run the code in the block.

The code in the block is even more relatable. We know of the counter from above and we can see 
that this increments the counter at every cycle of the loop. 
What we have here is known as a binary counter and it basicall does something along the lines of 

Bit 0 toggles every clock cyle 

Bit 1 toggles every 2 cycles 

...

Bit 27 toggles every 2^27 Cycles 

As we go higher in bits the clock take longer to toggle and this enables us to create visible effect 
later with the LEDs we will see this. 

In hardware terms this is called a `flip-flop` register that updates on the rising clock edge.

How is this magic its just a loop? Well in software this all happens sequentially you would update 
each significant bit at each cycle, In hardware each of the bits updates at once, this is why 
we use `<=` which is known as the non blokcing operator. It ensure all 28 bits read the old values
before updating together.

### More Magic. 
Ok now we have a counter! cool! Let's use it. Thus we encounter the lines below 

```v
SB_RGBA_DRV RGB_DRIVER (
    .RGBLEDEN(1'b1                                            ),
    .RGB0PWM (frequency_counter_i[25]&frequency_counter_i[24] ),
    .RGB1PWM (frequency_counter_i[25]&~frequency_counter_i[24]),
    .RGB2PWM (~frequency_counter_i[25]&frequency_counter_i[24]),
    .CURREN  (1'b1                                            ),
    .RGB0    (led_green                                       ), 
    .RGB1    (led_blue                                        ),
    .RGB2    (led_red                                         )
  );
```

Again looks very arcane, but we can understand this if we try 

`SB_RGBA_DRV RGB_DRIVER` the last time we encountered the `SB_` prefix it happened to be a 
primitive predefined in the FPGA this time its the same this is a RGBA driver that is 
in the fabric that we can use quickly, how amazing a stepping stone you could call it. 

Infact interestingly, `SB` means silicon blue and for the `Lattice iCE40` family this means 
a predefined primitive that we can use without defining!

so the line above assigns a name to the resident RGBA driver so we can use it. The we go further to 

`.RGBLEDEN(1'b1)` we set the enable pin of the RGB  druver to logic high

Then the following three lines allow us to setup prebuild PWM signals and wehen they are active

```v
.RGB0PWM (frequency_counter_i[25]&frequency_counter_i[24] ),
.RGB1PWM (frequency_counter_i[25]&~frequency_counter_i[24]),
.RGB2PWM (~frequency_counter_i[25]&frequency_counter_i[24]),
```
so what is happening here is:

When bit 25 and 24 of the binary counter are set the RGB0 (green) is on 

When bit 25 is set and bit 24 is reset then RGB1(blue) is on 

When bit 25 is reset and bit 24 is set  then RGB2 (red) is on

btw you can surmise these connections from [this guide](https://upduino.readthedocs.io/en/latest/features/specs.html)

`.CURREN(1'b1)` enables the current to the LEDs and must be set to a logic high for them to work.

The next three lines are even more sane, remember the outputs we defined at the top of the module 
we connect them to the LED instances  here 

```
    .RGB0    (led_green                                       ), //Actual Hardware connection
    .RGB1    (led_blue                                        ),
    .RGB2    (led_red                                         )
```

Perfect thats demystified lets  move on. 

Finally we see these 4 lines 

```v
  defparam RGB_DRIVER.RGB0_CURRENT = "0b000001";
  defparam RGB_DRIVER.RGB1_CURRENT = "0b000001";
  defparam RGB_DRIVER.RGB2_CURRENT = "0b000001";

endmodule
```

Firstly at the end is a keyword `endmodule` this concludes our top module i.e our app. 
Then we have 3 repeated lines 

`defparam RGB_DRIVER.RGB0_CURRENT = "0b000001";` defparam is a keyword that allows us to 
set a paramter of an instance after it has been instantiated. 
Here we are setting parameters of the RGBA driver that we have made up and for each of them 
we are setting the current driver to the lowest non zero value we can. In  other words we are setting the brightnes of the LED!

And that's it! we grokked our first FPGA module and now we have ideas and questions!


## References 
[LED Driver Guide](https://workshop.fomu.im/en/latest/_static/reference/FPGA-TN-1288-ICE40LEDDriverUsageGuide.pdf)










