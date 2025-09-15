#  Filtering Latency - Encountering a Guardian of the Threshold.


A [Guardian of the Threshold](https://en.wikipedia.org/wiki/Guardian_of_the_Threshold) is a 
menancing figure, which manifests itself as soon as 
``the student of the spirit ascends upon the path to higher worlds of Knowledge``.

In our journey of DSP there will be many of these, today we deal with Filtering Latency. 

[All filters come with latency](https://www.reddit.com/r/DSP/comments/mxnicm/comment/gvqaaw8/) 

You see previusly in our code we tried to filter the samples as we received them, 
but alas you can't do this, it takes time to filter and while you are filtering the 
audio still needs to be sampled!

Here is how we [are doing it](https://github.com/Gaiaocho/zephyr_custom_ble_service/pull/10/commits/80e9d8458c8e11cdb43de6a1a8dd340c30545c7d#diff-4d1a1063f29ab6d1c2144637e7acc6dc5ded655ecbafa222496ff0f1c4436880R129) 

This however ends up crashing our app!

```sh 
[00:05:05.874,938] <wrn> ble_module: Connection estalished!
[00:05:05.874,969] <wrn> fsm_module: INCOMING EVENT 0x00000001
[00:05:05.875,274] <wrn> fsm_module: Go to connected State
[00:05:05.875,274] <wrn> fsm_module: Left Idle
[00:05:05.875,274] <wrn> fsm_module: Entering Connected state
[00:05:05.875,366] <wrn> audio_module: SENSING AUDIO
[00:05:05.979,370] <wrn> audio_module: Consumed Samples
[00:05:06.173,034] <err> i2s_nrfx: Failed to allocate next RX buffer: -12
[00:05:06.364,624] <wrn> audio_module: Filtered a Block
[00:05:06.369,384] <wrn> audio_module: Consumed Samples
[00:05:06.756,347] <wrn> audio_module: Filtered a Block
[00:05:06.761,108] <wrn> audio_module: Consumed Samples
[00:05:07.145,751] <wrn> audio_module: Filtered a Block
[00:05:09.145,904] <wrn> audio_module: SENSING AUDIO
[00:05:09.249,908] <wrn> audio_module: Consumed Samples
[00:05:09.638,824] <wrn> audio_module: Filtered a Block
[00:05:09.643,798] <wrn> audio_module: Consumed Samples
[00:05:09.741,271] <err> i2s_nrfx: No room in RX queue
[00:05:09.741,271] <err> i2s_nrfx: No room in RX queue
[00:05:10.031,982] <wrn> audio_module: Filtered a Block
[00:05:10.037,017] <wrn> audio_module: Consumed Samples
[00:05:10.424,774] <wrn> audio_module: Filtered a Block

```

As you can see the RX Queue ends up running out of memory, and this leads to a crash. 
Intuition says the RX queue is filling up because of the delay between 
filtering and receiving 

- Can we pass filtering to another thread and handle it there?
- Do we need to mess with priorities between filtering and sensing?

 can we use the state machine, to transition from capturing audio to filtering audio.
 

## Plan 
1. Add a filtering Thread, with its own MEM_SLAB
2. copy block samples to filter blocks 
3. Apply the filter in our new thread
4. Use hysteresis detector in filter thread to maintain noise/ no noise state 
5. Do inferece in filter thread.


### Questions
1. How can we ensure that we do Inference right after  recording? FSM perhaps?
2. What sort of deadline can we adhere to with the Inference?
3. Is there anyway of making this faster?


