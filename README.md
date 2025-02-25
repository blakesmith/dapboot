# dapboot
The dapboot project is an open-source USB [Device Firmware Upgrade](http://www.usb.org/developers/docs/devclass_docs/DFU_1.1.pdf) (DFU) bootloader for STM32 devices.

Currently, the only targets officially supported are the STM32F103x series.

## Build instructions
The default target is a generic STM32F103 dev board with an LED on PC13, commonly referred to as a "bluepill" board.

To build other targets, you can override the
`TARGET` variable when invoking `make`.

    make clean
    make TARGET=STLINK

### Targets

| Target Name | Description | Link |
| ----------- | ----------- |----- |
|`BLUEPILL`   | Cheap dev board | https://stm32duinoforum.com/forum/wiki_subdomain/index_title_Blue_Pill.html |
|`MAPLEMINI`  | LeafLabs Maple Mini board and clone derivatives | https://stm32duinoforum.com/forum/wiki_subdomain/index_title_Maple_Mini.html |
|`STLINK`     | STLink/v2 hardware clones | https://wiki.paparazziuav.org/wiki/STLink#Clones |
|`OLIMEXSTM32H103` | Olimex STM32-H103 | https://www.olimex.com/Products/ARM/ST/STM32-H103/ |
|`BLUEPILLPLUSSTM32` | Bluepill with USB C | https://github.com/WeActTC/BluePill-Plus/ |

For each of the above targets, there are three variants that can be added to the target name:

| Target Variant | Description                                           |
| -------------- | ----------------------------------------------------- |
|` `             | Standard bootloader, using first 8kB of flash         |
|`_HIGH`         | High memory bootloader for 64kB chips  (experimental) |
|`_HIGH_128`     | High memory bootloader for 128kB chips (experimental) |

The high memory bootloader is a variation that doesn't require the application to be at an offset, the bootloader resides in the top 6.5kB of ROM and hides its reset and stack vectors inside unused entries of the application vector table. As an example, to compile for a Bluepill board with 128kB flash, use:

    make clean
    make TARGET=BLUEPILL_HIGH_128


## Flash instructions
The `make flash` target will use openocd to upload the bootloader to an attached board. By default, the Makefile assumes you're using a [CMSIS-DAP](http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php) based probe, but you can override this by overriding `OOCD_INTERFACE` variable. For example:

    make OOCD_INTERFACE=interface/stlink-v2.cfg flash

## Overriding defaults
Local makefile settings can be set by creating a `local.mk`, which is automatically included.

Here is an example `local.mk` that changes the default target to the STLink/v2 and uses an unmodified STLink/v2 to flash it.

    TARGET ?= STLINK
    OOCD_INTERFACE ?= interface/stlink-v2.cfg

You can also use the env variable `DEFS` to override default configuration for a target, like:

    # Disable LED on BluePill
    DEFS="-DHAVE_LED=0" make TARGET=BLUEPILL

    # Allow access to all Flash on MapleMini and change the app base address
    DEFS="-DFLASH_SIZE_OVERRIDE=0x20000 -DAPP_BASE_ADDRESS=0x08004000" make TARGET=MAPLEMINI LDSCRIPT="/some/folder/stm32f103x8-16kb-boot.ld"

## Using the bootloader
### Building for the bootloader
The standard bootloader occupies the lower 8KiB of flash, so your application must offset its flash contents by 8KiB. This can be done by modifying your linker script or flags as appropriate.

The high memory bootloaders do not use the lower part of the flash, so you only need to make sure your application leaves 6.5kB of flash free.


### Switching to the bootloader
The bootloader can be built to look for arbitrary patterns, but the default for the STM32F103 target looks for a magic value stored in the RTC backup registers. Writing the magic value and then resetting will run the bootloader instead of the main application.

The bootloader currently looks for `0x544F` in RTC backup register 1 and `0x4F42` in RTC backup register 0 (together they spell "BOOT" in ASCII).

You can also use a button to stay in bootloader while booting. It's configured using `HAVE_BUTTON` define. If your button has a debounce capacitor, you can use `BUTTON_SAMPLE_DELAY_CYCLES` define to specify how many cycles to wait before sampling the I/O pin, by default it is approximately 20ms in a 72Mhz MCU.

On the bluepill boards, the default is using the BOOT1 input (available on jumper in the board) to enter the bootloader.

### WebUSB
This bootloader implements the draft [WebUSB](https://wicg.github.io/webusb/) specification, which allows web pages to access the bootloader (after presenting the user with a device picker dialog).

For a demo implementing dfu-util features in the browser, see https://devanlai.github.io/webdfu/dfu-util/

To customize the WebUSB landing page, you can use the `LANDING_PAGE_URL` define. To set it from the command line, you can use the `DEFS` environment variable:

    DEFS='-DLANDING_PAGE_URL=\"example.com/dfu-util/\"' make

Note that the URL scheme shoul not be part of the `LANDING_PAGE_URL` string. As of this writing, it is hardcoded to HTTPS. 

## USB VID/PID
The default USB VID/PID pair ([1209/DB42](http://pid.codes/1209/DB42/)) is allocated through the [pid.codes](http://pid.codes/) open-source USB PID program.

To use a custom VID/PID pair, you need to set the macros `USB_VID` and `USB_PID`. One way to do this is by setting the `DEFS` environment variable when compiling:

     DEFS="-DUSB_VID=0x1209 -DUSB_PID=0xCAFE" make


## Licensing
All contents of the dapboot project are licensed under terms that are compatible with the terms of the GNU Lesser General Public License version 3.

Non-libopencm3 related portions of the dapboot project are licensed under the less restrictive ISC license, except where otherwise specified in the headers of specific files.

See the LICENSE file for full details.
