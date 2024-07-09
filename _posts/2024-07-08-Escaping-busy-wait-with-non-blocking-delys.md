 # Escaping Busy-Wait with Non blocking Delays. 


 When I start out working on a project most of the time I am concerned about 
 getting some hardware working or maybe some protocol being predictable. Lets 
 take a task such as say setting up the ADC, you need a small delay to give 
 the ADC time to ramp up. I usually do something like the following.

## Come on Wait Faster.


```c
void delay(void){
	for(uint32_t i = 0; i < 500000/2; i ++);
}
```


If you couldn't tell from the name of the function, this is delay function albeit 
not a very good one, to know why we have to explore how it works.

- This function setls up a loop that runs for a number of cycles in this case 
250,000
- The loop body is empty, so we can see this function does nothing but just 
count.
- Basically what we have achieved here is what is known as busy-wait loop, we 
are consuming cpu cycles so sometime can pass before we continue.


Here is an example of how we would use this function, let's say we have a LED 
on pin 15 of GPIOD, which the STM32F407 has as a Blue LED. After setting it up
we can do the following.

```c
GPIOD->BSRR |= GPIO_BSRR_BS_15;  // turn on the LED

delay(); // wait for sometime

GPIOD->BSRR |= GPIO_BSRR_BR_15; // turn off the LED
```


Now for very basic situations this, works especially if you are not concerned
about power consumption, timing precision and predictability, However it is 
hard to find yourself in this situation. Normally one would care about atleast 
one of these points.



## Efficiency, Precision and Predictability

Our code above is not efficient at it basically does work when it should be 
waiting, Imagine a parent driving their child to a new city and the child kept 
asking `Are we there yet?` every minute of the way! This would get very tiring
and distracting after a very small amount of time.

Precision, the way our code delays above is by wasting cpu cycles, which are 
the basic time units when cpus process instructions, in our case here the instruction 
is `increment i`. CPus run at certain speed called a clock speed and depending 
on what this is the delay above could be longer or shorter and thats hardly precise.

Predictability, if you squint at the paragraphs above you start to see that this 
not a delay you can rely on because you really can't tell what it would do.




## Solutions
We need to take a step back and see how we can solve this, I like looking at the
human world when I can for motivation. Humans usually `process` delays by either
doing an idle wait consider sleeping or by actively doing something else and 
checking the time to check if they should go back to what thet are delaying.

In either case here we have a concept of time vis a vis when is it now? and when 
will it be when I have finished waiting. 

If it `9:05` and I need to wait for 8 hours I could just sleep till `17:05`

==We need to teach our system about time! :bulb: ==


### Introducing SysTick
Since I am using a STM32F407 based on ARM I will introduce ARMs version of this.

Basically this is a system timer that can be configured to generate regular 
interrupts. 

Say we have a CPU running at 2MHz we can generate an interrupt every 500 milliseconds 
by scaling it down using the following formula

If the frequency \( f \) is 2 Hz, the period \( T \) is:

\[ T = \frac{1}{2 \text{ Hz}} = 0.5 \text{ seconds} \]

This means that each cycle takes 0.5 seconds to complete.


We can use this formula to generate any length cycle we want.


Now out system has a concept of  time and its passing! this is really the bulk 
of the work.

From here we can create and manage simple delays by getting the current tick, 
calculating how many ticks need to pass before we  consider our delay done. 

```c 
GPIOD->BSRR |= GPIO_BSRR_BS_15;
			

while(!Timer_IsElapsed(&mainTimer)) {
	//we should yield to other tasks here ...	
	__WFI(); // Let's sleep we can be interrupted but will wait
}
			
int8_t today_temp = Lis3ReadTemp();
printf("Temp is %dC Degrees Celsius \n", today_temp);
```

And inadvertently by fixing the first issue we know about we have fixed the 
by using  a hardware timer we have predictable running times, by structuring our 
code as above we can say `sleep` as we wait we are not efficient and we can be 
sure the system will react the same everytime. 

Obviously my solution above uses code that we I did not introduce or explain, the
intention here is that the reader should be able to take these instructions and 
use them as they deem fit in there system.

If however you do need to see this work do look at the [Full Codebase](https://github.com/zacck/uCDSP/tree/main/Src)



Thank you for reading.!

