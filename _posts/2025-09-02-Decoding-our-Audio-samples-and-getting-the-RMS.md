# Decoding Audio Packets and Performing Basic Signal Processing 

## How much Audio do we have?

```c
#define SAMPLE_FREQUENCY 	44100
/*Mic data sheet*/
#define SAMPLE_BIT_WIDTH 	24
/*24 Bits packed into a 32 bit block*/
#define BYTES_PER_SAMPLE 	sizeof(int32_t)
#define NUMBER_OF_CHANNELS 	1
/*4410 samples so 0.1s*/
#define SAMPLES_PER_BLOCK 	((SAMPLE_FREQUENCY / 10) * NUMBER_OF_CHANNELS)
#define INITIAL_BLOCK_COUNT 	1
#define TIMEOUT_MS 		2000

/*Because i2s_read expects number of bytes*/
#define BLOCK_SIZE 		(BYTES_PER_SAMPLE * SAMPLES_PER_BLOCK) 
/*Enough for 400ms of sound*/
#define BLOCK_COUNT		(INITIAL_BLOCK_COUNT + 3)
```

Like the comments suggest above we are reading 4 byte words but we only care about the 3 most 
significant byte i.e 24 bit data. That means if we have a word like `0x12345600` our actual
data is `0x123456` and the last byte is just padding to make life easier.

- At sample frequency of `44100` which is the number of samples in a second.
- Our Memory block is sized at a 10th of that, so `4410` samples and we have 3 of these blocks. 
- Simple math says we collect around 4 tenths of a second or in human terms `400ms` of audio. 

### How come this amount?

Why `400ms` well really its a arbitrary but eventually we want to do keyword recognition and 
this is around enough time for the fitting of a phrase like `Yo Machine!`. 

That is why we chose this amount of Audio.


### Decoding the Audio
 Remember above when we spoke about the Audio word structure, well if you have 24 bit audio in 
 in a 32 bit word 

 ``` 
 MSB                        LSB
 | 24 bit audio| 8 bit padding|
 ```
Here is how  every sample would look like. Lets get the value out of it 

1. We need to discard the padding 
2. We need to normalize the float 
3. Then we can print it or something so we we see it

the result would look like 

```c
for(int i = 0; i < 100; i++) {
    int32_t sample_five = samples[i];
    sample_five = sample_five >> 8;
    float normalized_five = sample_five/8388608.0f;
    printf("%f \n", (double)normalized_five);
}
```

And voila! we have our audio sample now we can convert this into decibels or whatever unit 
we want, for our application however this is not necessary what we want to do at this stage
is to just detect some sound so basically we need to check if some `threshold` has been 
crossed by the `amplitude` of the signal. 

Ok however remember we are recording `44100` samples per a second, if we attempted to detect noise
or silence for each sample, the data would be hardly meanigful the sample times are too short 
and the calculations are too expenive. Let's look at each of our memory blocks above, with each of 
them having `4410` samples, roughly a 100ms time this starts to become manageable we can use some 
statistics techniques and compute a mean of the value 100ms block. 

That should surely work! Yes, but not really you see sound isnt that simple often we are using 
electronics to record sound. They have a large sensitivity range and are sensitive to thing like 
electro magnetics. This is to say sound has a few components in it and when we are measuring it 
we need to account for them. 

Things such as
1. Background noise
2. EM interference
3. Actual sound we want
5. etc 

In comes the `Root Mean Square`, this is a measure in statistics that takes account for 
the DC and the AC elements in our signal. Simply defines the RMS of a set of values 
is the square root of the set's mean square also known as `variance`, the RMS is also 
known as the quadratic mean.

$$
\text{RMS} = \sqrt{\frac{1}{N} \sum_{i=0}^{N-1} x_i^2}
$$

This enables us to account for active audio and Background noise. 




### References

[Signal and Graph Terms](https://www.dspguide.com/ch2/1.htm)
[Root Mean Square](https://www.dspguide.com/ch2/2.htm)
[Numbers in Memory](https://www.tutorialspoint.com/fixed-point-and-floating-point-number-representations)
[Amplitude Threshold](https://dsp.stackexchange.com/questions/25446/algorithm-for-detecting-the-time-where-the-signal-is-above-a-threshold)


