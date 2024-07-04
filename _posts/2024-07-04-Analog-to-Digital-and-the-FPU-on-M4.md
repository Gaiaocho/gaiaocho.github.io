# Using the Analog to Digital Converter with the FPU on the Cortex M4



I intend for today discussion to be short and sweet. I am doing this as a body to 
prepare for some digital signal processing. However, today I will tackle a small
part of this. Just reading float values from the ADC  on the Cortex M4, my
Microcontroller of choice will be the STM32F407.


## Objectives 
1. Configure an ADC peripheral to Accept input on a channel.
2. Configure the Floating Poiunt Unit on the M4 for floats.
3. Consume the input for that channel using floating point calculations.
4. Log out the results of the calculations.

## Coding Session 
You can also follow this on my coding session [here](https://youtube.com/live/AzGZ5jSR6TI)


What the above will show is how to set up your ADC to accept Analog input 
and convert it into meaningful data. 

Then how to use this meaniful data(voltage) in your application sometimes
involving floating points.

Finally what settings you will need to use to enable the FPU



### Configuring the ADC.
So In order to use the one of the ADC channels we need to do the following

- Enable the clock for the ADC
- Choose which Resolution we will use
- Choose a conversion mode, one shot or scan
- Choose which channel to use 
- Set a Sample rate for the channel
- Start the ADC
- In our case we will enable the internal temp sensor so we can read from it.

Lets do the above with the following code for ADC1, 12 bit resolution, Single 
Conversion, Channel 16, at 490 cycles sample rate.

```c
// Configure ADC so we accept one in input on one ch
	//enable ADC1 clock
	RCC->APB2ENR |= (1 << RCC_APB2ENR_ADC1EN_Pos);
	//choose 12 bit resolution
	ADC1->CR1 &= ~(3 << ADC_CR1_RES_Pos);
	// conversion mode, single default
	ADC1->CR2 &= ~(1  << ADC_CR2_CONT_Pos);

	// set channel to be read ADC1_IN16
	ADC1->SQR3 &= ~(0x1F);
	uint32_t channel_number = 16;
	ADC1->SQR3 |= (channel_number  & 0x1F);

	// Sample Rate
	ADC1->SMPR1 |= (7 << ADC_SMPR1_SMP16_Pos);

	//Enable temp sensor
	ADC->CCR |= (1 << ADC_CCR_TSVREFE_Pos);

	//Start ADC1
	ADC1->CR2 |= (1 << ADC_CR2_ADON_Pos);

	// give ADC and Temp Sensor time to ramp up
	delay();
```

### Configure the FPU on the STM32F407
Now we have our peripheral working and we can measure Channel 16. In order to 
convert our raw data into voltage, we need to do some calculations see them 
below


```c
	float Vsense = (float)temperature_raw/4096 * 3.3;

	float VDeg =((Vsense - 0.76)/0.25) + 25;
```

These need us to use Floating point arithmetic and we need to enable this on 
our chip as follows.


```c
	SCB->CPACR |= ((3UL << 20U)|(3UL << 22U));
```

This plus  the compoliler flags below enable FPU 

```
--gc-sections -static --specs=nano.specs -mfpu=fpv4-sp-d16 -mfloat-abi=hard
-mthumb -u _printf_float -u _scanf_float -Wl,--start-group -lc -lm -Wl,--end-group

```



### Consume data and Print it out
Since we have the FPU activated and the raw measurement available lets combile 
all these into some meaningful information


```c
    // Read input and print it
	ADC1->CR2  |= (1 << ADC_CR2_SWSTART_Pos);

	// wait for end of conversion
	while(!(ADC1->SR & (1 << ADC_SR_EOC_Pos)));

	//read adc_raw_input
	uint16_t temperature_raw = ADC1->DR;

	printf("Some Values from ADC COnversion %d \n", temperature_raw);
	// do floating point arithmetic on reading of input
	// print the above

	//enable FPU
	SCB->CPACR |= ((3UL << 20U)|(3UL << 22U));

	float Vsense = (float)temperature_raw/4096 * 3.3;

	printf("Raw value to Voltage from ADC COnversion %1.1f \n", Vsense);

	float VDeg =((Vsense - 0.76)/0.25) + 25;

	printf("Raw Voltage from ADC to Temp in Degrees Celcius: %1.1f \n", VDeg);

```

Effectively above we read the value from the ADC channel use FPU math to do some
calculations and print it out here's how it looks on my system 


```
System booted with Default clock
Some Values from ADC COnversion 1017 
Raw value to Voltage from ADC COnversion 0.8 
Raw Voltage from ADC to Temp in Degrees Celcius: 25.2
```


There you go, you have ADC working with the FPU.





