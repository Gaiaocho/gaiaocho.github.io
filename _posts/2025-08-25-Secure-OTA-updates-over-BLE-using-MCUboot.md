# OTA Updates with MCUboot

### MCUBoot the Basics
- Started out as the bootloader for Mynewt and Apache RTOS. 

To use MCUBoot we need the following partitions
    a) boot_partition: for MCUboot
        How to add partitions over DTS?
    b) slot0_partition: the primary slot of image 0 
    c) slot1_partition: the secondary slot of image 0
 Although it is not recommended one can perform swap-using-scratch using 
    d) scratch_partion: the scratch slot

- Image slots 0 & 1 need to be contigous.
- If we are using MCUboot as the first stage BL then, the MCU must be 
configured to run `boot_partition` out of reset. 
- If there are multiple updated images then the corresponding primary and 
secondary slots must be defined for the images. 

**Flash partitions are usually defined in the dts and can be extended in overlays**
**MCU boot might need extra package installation before use**

#### Building MCUboot
- MCUBoots acts as a normal application to zephyr, as all [bootloaders are applications 
with special permissons really](https://github.com/zacck/stm32f4_uart_bootloader_app).
- As such it needs to be configured and built as well, there are instructions in the 
`/boot/zephyr` directory of MCUboot on doing this 
- All configuration used is in `boot/zephyr/include/target.h`
- After building we will have a binaries in the usual zephyr build folder.
- We can then flash the bootloader to the device. 


#### Building applications for MCUboot. 
- Along with the flash modifications we need to configure apps for MCUboot. 
- This is handled internally by zephyr using [CONFIG_BOOTLOADER_MCUBOOT](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_BOOTLOADER_MCUBOOT)
- Checkout [sysbuild in Zephyr](https://docs.zephyrproject.org/latest/samples/sysbuild/with_mcuboot/README.html)

#### Signing Applications
- A customary step in the bootloader is to check the [checksum of the application](https://github.com/zacck/stm32f4_uart_bootloader_app/blob/1cbbf9fd524b34892fb7f42df651e216e6ac3ae9/Core/Src/main.c#L151) additionally along with verification we can and should sign applications with keys which MCUboot requires. 
Zephyr provides `scripts/imgtooll.py` script to aid us with making our own signatures.


### Understand DFU (Device Firmware Update) flow
Applications built this way can be flashed the normal way and if using the intex Hex format 
they dont need much  but some care has to be taken for the following items

- One may need to select an application offset, this goes to the images primary slot. 
- One needs to make sure the flasher does not perform a mass erase as this will get rid
of MCUboot. 
- Images can also be marked for upgrade and loaded into the secondary spot, if this occurs then
the bootloader should perform an upgrade.
- The image is then expecte to mark the primary slot as `image-ok` before the next reboot, otherwise the bootloader will revert the application.

Summarily
1. MCUboot will boot first and verify the integrity and authenticity of the image in the primary slot.
2. If it detects a valid new image in the secondary slot, 
    a) using trailer flags `IMAGE_OK`, `COPY_DONE`,`BOOT_PENDING`
    b) It may compare versions
    c) If the secondary image is valid and is in `BOOT_PENDING` it is made read for swap
    Zephyr's DFU subsystem  manages the trailer flads for us this is [MCUmgr](https://docs.zephyrproject.org/latest/services/device_mgmt/mcumgr.html)
3. Also see [swap using scratch](https://docs.mcuboot.com/readme-zephyr.html)

### Serial Recovery aka Don't Brick the Device
Use this to load new firmware explicitly. 

- Serial recovery is available over `USB CDC ACM` hardware or virtual port.
- This enables a SMP host  to connect and communicate with the device 
- choose a port using `BOOT_SERIAL_DEVICE` using the chosen node. 
- To enter serial recovery the deice has to initiate rebooting [recall wake sources](https://gaiaochos.com/2025/08/21/NRF52840-Power-management-and-Radio.html)
- Zephyr has a `mcuboot_buttn0`! 
- MCUboot can also have a timeout set to listen for a MCUmgr command.
- Once connection has established the SMP server can then use the first slot then can invoke 
Image upload
 
--- 

### Implementation
- `CONFIG_MCUBOOT`: signifies that the target uses MCUboot.
- `CONFIG_IMG_MANAGER`: enables DFU image management in zephyr.
- `CONFIG_MCUMGR`: enables mcumgr library that provides DFU
- `CONFIG_MCUMGR_TRANSPORT_BT`: Enables BLE transport for mcumgr.
when using sysbuild we add conf file for it `sysbuild.conf`
`SB_CONFIG_BOOTLOADER_MCUBOOT` allows us to build our application with MCUboot in one go.
- this negates the need to build separately and the need for `CONFIG_MCUBOOT` in our .conf 

#### CODE
- Adding OTAtep-by-step plan for adding OTA to your project:
  1. Enable MCUboot and mcumgr in `prj.conf`.
  2. Build and flash the bootloader.
  3. Add SMP service over BLE in your app.
        a) include `#include <mgmt/mcumgr/smp_bt.h>`
        b) after ble init call `smp_bt_register()` before we advertise
  4. Test uploading a new image with `mcumgr`
  - connect and upload
  ```sh
  # connect
  mcumgr --conntype ble --connstring ctlr_name=hci0,peer_id=XX:XX:XX:XX:XX:XX image list

  # upload
  mcumgr --conntype ble --connstring ctlr_name=hci0,peer_id=XX:XX:XX:XX:XX:XX image upload \  
  build/zephyr/zephyr.signed.bin
  
  ```
  
  Then to test and reset

  ```
  mcumgr image test <hash>

  mcumgr reset
  ```

 Summary:**“How would you implement safe OTA?”**
  - Require bonded/encrypted connection before DFU.
  - Verify image signature (signed images).
  - Allow rollback if update fails.

