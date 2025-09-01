# Debugging I2S Memory issues

For a few days  now I have had this issue where my audio thread's i2s peripheral ran out 
of memory below are a few things I have tried. 

### Checks
- Microphone data format 
- Microphone alignment of bits 
- How to use Mem Slabs
- How much time do out Mem Slabs hold
- Check I2S State 


## The Issue! 

Turned out to be how I was invoking `k_mem_slab_free()`, you see zephyr expects a `(void *)`
for the second argument of the function and I was passing in a pointer without casting it, t
this caused that function to fail quietly which led to me running out memory blocks 

```diff

void *mem_block; 
....
int32_t *samples = mem_block;
LOG_WRN("Sample number 5 in the Block Samples: %d", samples[5];
- k_mem_slab_free(&mem_slab, mem_block); 
+ k_mem_slab_free(&mem_slab, (void *)mem_block);

```

Turns out this simple change is all I needed and I needed it even though the pointer was a `void *`
I guess this is like the thing they say about infinities, some infinities are bigger or smaller than 
others,  ...


Links
[zephyr mem slabs](https://docs.zephyrproject.org/latest/kernel/memory_management/slabs.html)

[i2s](https://hackaday.com/2019/04/18/all-you-need-to-know-about-i2s/)

[old zephyr i2s](https://docs.zephyrproject.org/2.7.5/reference/audio/i2s.html)


