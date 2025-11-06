# FRDM-MCXN236 and OV5640

I've had trouble with this set up before, seems to me everytime I try and set up a board
the first time it never works. However when I come back for another pass things are usually 
very straight forward, today my experience with the [MCXN236](https://docs.zephyrproject.org/latest/boards/nxp/frdm_mcxn236/doc/index.html) has been the same. 

Perhaps I was in a rush, perhaps I need more patience?

Regardless, let's actually explore. My objective today is to hook up a camera to the MCXN236
specifically the [OV5640](https://www.adafruit.com/product/5673). Then try and see if we can 
achieve some UVC video as well, see if we can use [libmpix](https://libmpix.org/) to do 
some signal processing on the videos themselves. 

Let's begin. 

## Setup
To start with lets get the board and try and run some zephyr samples on it. 

1. Install and setup LinkServer tools as linked in the Zephyr Board Page
2. Build and flash a basic blinky application onto the board
```sh
west build -b frdm_mcxn236 zephyr/samples/basic/blinky --pristine
...
west flash 
```
3. The above should give you a red blinking, not very exciting, I know! 
4. Acquire the OV5640 Camera module and have it ready, infact you can connect it up 
the board as follows, connecting the camera should light up the green LED on the back of the
camera module. This means `power good`. 

![Board and Camera Connected](/assets/nxp_ov5640.jpg)

Ok so thats our hardware all setup, let's see if we can configure the camera and get some 
captures from it. 


## Overlays & Configuration
In order for this to work with as little effort as possible lets first ensure that a few
items exist. 

1. A video driver for this MCU and indeed we have a [SDMA Driver](https://docs.zephyrproject.org/latest/build/dts/api/bindings/video/nxp%2Cvideo-smartdma.html#std-dtcompatible-nxp-video-smartdma)
2. A driver for our image sensor module [OV5640 Driver](https://github.com/zephyrproject-rtos/zephyr/blob/main/drivers/video/ov5640.c)

With these 2 now the issue becomes plumbing the data from the image sensor via sdma to 
some chosen `zephyr,camera` node, this should enable us to get the video subsytem working

3. Setup the camera itself on the `dvp_i2c` block as follows 

```c 
&dvp_20pin_i2c {
	ov5640: ov5640@3c {
		compatible = "ovti,ov5640";
		reg = <0x3c>
		#reset-gpios
		#power-down-gpios

		port {
			ov5640_ep_out: endpoint {
				remote-endpoint-label = "ov5640_sdma_ep_in";
			};
		};
	};
};

```
   

4. Send the camera data from the `dvp_i2c` block to the `dvp_sdma` block  as follows 

```c
&dvp_20pin_interface {
	status = "okay";

	port {
		ov5640_sdma_ep_in: endpoint {
			remote-endpoint-label = "ov5640_ep_out";
		};
	};
};
```

5. Set the chosen `zephyr,camera` node.

```dts
#include <zephyr/dt-bindings/video/video-interfaces.h>

/ {
	/*Chosen cam node*/
	chosen {
		zephyr,console = &flexcomm0_lpuart0;
		zephyr,shell-uart = &flexcomm0_lpuart0;
		zephyr,camera = &dvp_20pin_interface;
	};
};

```
 

~~### TODO: File an issue and likely work on a PR for this.
This all currently works, however it fails with the following when it is time to capture~~

```sh
*** Booting Zephyr OS build v4.2.0-4812-g48a9b04885cb ***
[00:00:00.125,000] <inf> main: Video device: video-sdma
[00:00:00.125,000] <inf> main: - Capabilities:
[00:00:00.125,000] <inf> main:   RGBP width [320; 320; 0] height [240; 240; 0]
[00:00:00.125,000] <err> main: Unable to retrieve video format
```

We filed a PR and a fixed the above see a complete repository below.

Now one can expect the log below 
```sh
uart:~$
*** Booting Zephyr OS build v4.3.0-rc2-96-g8b2c75a2fbe4 ***
[00:00:00.126,000] <inf> main: Video device: video-sdma
[00:00:00.126,000] <inf> main: - Capabilities:
[00:00:00.126,000] <inf> main:   RGBP width [320; 320; 0] height [240; 240; 0]
[00:00:00.126,000] <inf> main: - Video format: RGBP 320x240
[00:00:00.140,000] <inf> main: - Supported frame intervals for the default format:
[00:00:00.140,000] <wrn> main: The video source does not support frame rate control
[00:00:00.140,000] <inf> main: - Supported controls:
[00:00:00.140,000] <inf> main:          device: ov5640@3c
[00:00:00.140,000] <inf> video_ctrls:                       Brightness 0x00980900 (int)    (flags=0x00) : min=-15 max=15 step=1 default=0 value=0
[00:00:00.140,000] <inf> video_ctrls:                         Contrast 0x00980901 (int)    (flags=0x00) : min=0 max=255 step=1 default=0 value=0
[00:00:00.140,000] <inf> video_ctrls:                       Saturation 0x00980902 (int)    (flags=0x00) : min=0 max=255 step=1 default=64 value=64
[00:00:00.140,000] <inf> video_ctrls:                              Hue 0x00980903 (int)    (flags=0x00) : min=0 max=359 step=1 default=0 value=0
[00:00:00.140,000] <inf> video_ctrls:                  Gain, Automatic 0x00980912 (bool)   (flags=0x10) : min=0 max=1 step=1 default=1 value=1
[00:00:00.140,000] <inf> video_ctrls:                  Horizontal Flip 0x00980914 (bool)   (flags=0x00) : min=0 max=1 step=1 default=0 value=0
[00:00:00.140,000] <inf> video_ctrls:                    Vertical Flip 0x00980915 (bool)   (flags=0x00) : min=0 max=1 step=1 default=0 value=0
[00:00:00.140,000] <inf> video_ctrls:             Power Line Frequency 0x00980918 (menu)   (flags=0x00) : min=0 max=3 step=1 default=1 value=1
[00:00:00.140,000] <inf> video_ctrls:                                  0: Disabled
[00:00:00.140,000] <inf> video_ctrls:                                  1: 50 Hz
[00:00:00.140,000] <inf> video_ctrls:                                  2: 60 Hz
[00:00:00.140,000] <inf> video_ctrls:                                  3: Auto
[00:00:00.141,000] <inf> video_ctrls:                    Analogue Gain 0x009e0903 (int)    (flags=0x0c) : min=0 max=1023 step=1 default=0 value=16
[00:00:00.141,000] <inf> video_ctrls:                       Pixel Rate 0x009f0902 (int64)  (flags=0x01) : min=24000000 max=96000000 step=1 default=48000000 value=24000000
[00:00:00.141,000] <inf> video_ctrls:                     Test Pattern 0x009f0903 (menu)   (flags=0x00) : min=0 max=4 step=1 default=0 value=0
[00:00:00.141,000] <inf> video_ctrls:                                  0: Disabled
[00:00:00.141,000] <inf> video_ctrls:                                  1: Color bars
[00:00:00.141,000] <inf> video_ctrls:                                  2: Color bars with rolling bar
[00:00:00.141,000] <inf> video_ctrls:                                  3: Color squares
[00:00:00.141,000] <inf> video_ctrls:                                  4: Color squares with rolling bar
[00:00:00.141,000] <inf> main: Capture started
```

#### Device Emulation Reference 

https://www.youtube.com/watch?v=usXCAXR2G_c

