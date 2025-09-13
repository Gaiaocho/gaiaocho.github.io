# Moving towards a RISC Implementation | Multiple GP Registers

Previously we modified our RGB Driver code to include a [very simple RISC style CPU](2025-09-12-Building-a-Tiny-CPU-FPGA.md). 
I would like to attempt to move this CPU to an actual RISC implementation and that means 
trying to adhertre to the [RISC Characteristics](https://en.wikipedia.org/wiki/Reduced_instruction_set_computer#Characteristics_and_design_philosophy) and of the first of these is that
 these computers habe many (16 or 32) high speed, general-purpose registers.

This and many more features, however today all I focusing on is actually adding and 
using multiple General Purpose registers. 

---

## Current State.
At the moment our computer only has one General Purpose Register 

```v
// CPU  General Purpose register
  reg [2:0] A;
```
Aptly named however my trusty STM32F4 has R0 to R15 and  I would like to emulate that! 

The way a RISC CPU handles having more than one instruction resgister is that it uses a 
[load-store architechture](https://en.wikipedia.org/wiki/Load%E2%80%93store_architecture)
this is where the computer has an instruction set with 2 categories. 
In this case 
- register - memory access: load from memory  store to memory
- register - register access: ALU operations that only happen in between registers

Using an example in such a system 
When performing an ADD instruction both operands should already be in registers  as opposed to a
[CISC](https://en.wikipedia.org/wiki/Complex_instruction_set_computer) architecture where one of the operands may be in memory and the other in a register. 

---

## OK Lets do it 

Firstly I would like to add 15 more registers that way we haver 16 so the code above becomes

```v 
reg [2:0] R [0: 15];
```

Then we have to update our code so that actually uses these registers. However this now introduces 
an issue, how do we know which register to use?

Recall!  We have a lot of unused space in our instructions

```v 
initial begin
	  ROM[0] = 8'b0001_0000; // go to next instruction
	  ROM[1] = 8'b0010_0000; // Output result of computation
	  ROM[2] = 8'b0001_0000; 
	  ROM[3] = 8'b0010_0000;
 	  ROM[4] = 8'b0001_0000;  
	  ROM[5] = 8'b0010_0000;  
	  ROM[6] = 8'b0001_0000;  
	  ROM[7] = 8'b0010_0000;  
	  ROM[8] = 8'b0001_0000;  
	  ROM[9] = 8'b0010_0000;
	  ROM[10] = 8'b0001_0000; 
	  ROM[11] = 8'b0010_0000;
 	  ROM[12] = 8'b0001_0000;  
	  ROM[13] = 8'b0010_0000;  
	  ROM[14] = 8'b0001_0000;  
	  ROM[15] = 8'b0010_0000;  
  end
```

We can see here we only use the first 4 bits, so one way of tackling this is to actually 
include the register address that contains our instruction!  Lets use the 4 LSB to encode
an address!


```v
initial begin
    ROM[0]  = 8'b0001_0000; // increment R0
    ROM[1]  = 8'b0010_0000; // operation on R0
    ROM[2]  = 8'b0001_0001; // increment R1
    ROM[3]  = 8'b0010_0001; // operation on R1
    ROM[4]  = 8'b0001_0010; // increment R2
    ROM[5]  = 8'b0010_0010; // operation on R2
    ROM[6]  = 8'b0001_0011; // increment R3
    ROM[7]  = 8'b0010_0011; // operation on R3

    ...
```

That fixes the issue we had now we know what to do and where to do it! 

--- 

## Putting all this together! 
OK now lets put all this together luckily this is simple! we already have a case statement 
and this only needs to be informed of the what and where.

It looks as such now

```v
always @(posedge slow_clk) begin
  	case(ROM[PC][7:4])
	  4'b0001: A <= (A + 1) & 3'b111; 
	  4'b0010: ;
	  default: ;
	endcase
	PC <= PC + 1; 
  end
```

This is actually most of the way there! all we need to is incase its an increment we need to know 
where to target and the line that changes is as below


```v
4'b0001: R[ROM[PC][3:0]] <= (R[ROM[PC][3:0]] + 1) & 3'b111; // increment
```

What is happening here?
If the instructions is to Increment

- Read the address of the instruction `ROM[PC][3:0]`
- Use the result of that to index into the R registers `R[ROM[PC][3:0]]`
- We perform an increment on that register masked by 111 `(R[ROM[PC][3:0]] + 1) & 3'111`
- This gives us a value that we set to `ROM[PC][3:0]`


That fixes it however we still need a way to observe this and formally we were using LEDs
lets still use them 

This is how the output code looked 

```v
.RGB0PWM (A[2] ),
.RGB1PWM (A[1] ),
.RGB2PWM (A[0] ),
```


But alas! we don't have an A register so what we need to do is check the current register in 
the instruction set! 

so this becomes 

```v
.RGB0PWM (R[ROM[PC][3:0]][2]), // MSB of selected register
.RGB1PWM (R[ROM[PC][3:0]][1]),
.RGB2PWM (R[ROM[PC][3:0]][0]), // LSB of selected register
```

Et Voila!  
We have multiple register and adhere to our first, RISC standard. Next time lets look at using the 
[OpenRisc instruction set!](https://en.wikipedia.org/wiki/OpenRISC)
