# Pinnacle
Open Source custom firmware replacement for the Puffco® Peak vaporizer.

The stock firmware of the device has several limitations, the most obvious of which are non-configurable Temperature Settings.

With the huge third-party accessory market manufacturing replacement buckets from Quartz, Silicon Carbide and other materials the limitations of the 4 built-in temperature modes are immediately apparent.

The goal of this project is to produce an Open Source replacement firmware for the device which first establishes feature parity with the Original Firmware, followed by integrating new features requested by the community such as user-configurable Temperature Setpoints and LED Control.

## Currently Implemented:
- Vibration Motor Control
- Power Button Control
- LED Control
- NTC Thermistor (Battery Temperature)
- UART Serial Port Output
- Battery Charge Detection

## To Do:
- Battery Voltage Detection
- Atomizer Heat
- Atomizer Resistance / Temperature Monitor
- General Program Flow

## Planned Features:
- nRF51822 BLE Module via On-Board Serial Port

## Programming Hardware:
The manufacturer has made it quite easy to start hacking on the device, and it does not require you to take apart your device to download or upload code.

The "Micro USB" port on the back of the unit provides a charging port for the battery, but does not actually expose a USB interface.

With a little bit of knowledge of the board layout and typical STM32F configurations, we were able to determine the D+ and D- lines on the Micro-USB port are actually a SWD (Serial Wire Debug) interface to the processor.

To make the custom programming cable, you will need the following items:

- ST-Link V2 (Can be found on eBay for about $5 USD)
- Micro-USB Cable

You will need to cut the Micro-USB cable and strip the wires to attach them to the ST-Link. We recommend attaching Dupont pin headers, though if a crimper is not available, you may solder jumper wires that are pre-crimped with dupont headers on one end. In the end, your final cable should look like the following:

![Custom cable](custom_cable.jpg)

Do not order premade cables as they all come crimped in an layout for ST-Link V2 pinouts. 

- STLink 5v -> USB Red Wire
- STLink GND -> USB Black Wire
- STLink SWD IO -> USB Green Wire
- STLink SWD CLK -> USB White Wire


## Development Toolchain Setup
This document assumes you are running on Mac OS or Linux.

You will need a Python3 environment and to pip install pyusb or libusb. We also assume you have a basic environment setup with the normal tools: homebrew, git, etc.

If you wish to do normal debugging and set breakpoints, etc. This document will assume you have VSCode installed.

Make sure your device is connected to the ST-Link with the custom USB to SWD Cable.

### Windows specific instructions

Windows requires a bit more configuration as it doesn't come with a development environment set up out of the box, however it works just as well with a bit of work. 

1. Download and install STM drivers for interfacing with the ST-Link from here (free account needed): [STSW-LINK009](https://www.st.com/en/development-tools/stsw-link009.html)
2. Install the firmware update utility for the ST-Link V2 from here: [STSW-LINK007](https://www.st.com/en/development-tools/stsw-link007.html)
3. Plug in the ST-Link V2 and load the firmware utility. Follow the directions to flash new firmware to the device. This is important as older firmware may not interface properly going forward with ST's various software. 
4. [Install Python 3](https://www.python.org/downloads/) and make sure PIP is installed
5. Open a terminal window (either using Windows Terminal or Command Prompt) and run the following:
`pip install pyusb`

NOTE: If you have more than one Python version installed you may need to specify what PIP version you want to use to install the packages. `pip3` is usually the most common nomenclature for the proper command in these instances but may not be depending on your system configuration. 

6. You will need to install libusb which on Windows consists of a single DLL and a LIB file. These are stored in the repository in the Windows_Dependency folder (credit to the [libusb](https://github.com/libusb/libusb) project). Copy `libusb-1.0.dll` to `C:\Windows\System32` and also to `C:\Windows\SysWOW64` (both locations are required on 64 bit systems). Then copy `libusb-1.0.lib` to `%LOCALAPPDATA%\Programs\Python\<VERSION>\libs` substituting "VERSION" with the installed Python version you are using for the project. 

NOTE: The supplied libusb files are 64 bit specific for now. There are 32 bit versions available from the above libusb project link.

7. (OPTIONAL): I recommend you install the [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) which can help set things like encryption flags on the processor, dump firmware and generally make working with STM chips easier. 

Continue as normal following the rest of the steps outlined below. 

### Backup OFW (Original Firmware)
You should take a dump of your original, unmodified device firmware before you go any further and keep it in a safe place.

- Verify MCU Connectivity

`pystlink -i` should return output similar to this:

```
DEVICE: ST-Link/V2 V2J33S7
SUPPLY: 3.24V
CORE:   CortexM0
MCU:    STM32F030x6/STM32F031x6/STM32F038x6
FLASH:  32KB
SRAM:   4KB-->
```

Pull down a copy of [PySTLink](https://github.com/pavelrevak/pystlink.git).

- Backup Device Flash: `pystlink read:flash:flash.bin`

- Backup Device SRAM: `pystlink read:sram:sram.bin`

If you ever wish to return your device to its original factory state, you will need to reflash this file back to the device and we cannot help you if you lose it.

### Compiler

You can install GCC for ARM, an STLink utility and a GDB debugger with these Homebrew commands

`brew tap nitsky/stm32`

`brew install arm-none-eabi-gcc`

`brew install --HEAD stlink`

### Debugging

If you wan't to open your device and wire up a TTL to Serial converter, you can send printf statements to the serial port.

But since we already have an STLink connection and we plan to use the serial port for the Bluetooth module / some people might not want to open their device, we can setup the real debugger.

- Section In Progress

### Flashing Firmware

You can erase and reprogram the STM32F Flash with a single command.

`pystlink flash:erase flash:file_to_flash.bin`

You will see output such as:

```
DEVICE: ST-Link/V2 V2J33S7
SUPPLY: 3.24V
CORE:   CortexM0
MCU:    STM32F030x6/STM32F031x6/STM32F038x6
FLASH:  32KB
SRAM:   4KB
Erasing FLASH: [========================================] done in 0.10s
Loaded 32768 Bytes from file_to_flash.bin file
Writing FLASH: [========================================] done in 1.57s
```

## MCU and Pinout:
The device is controlled by a ST Microelectronics STM32F030 Microprocessor

This graphic will be updated as development on the Custom Firmware progresses and the remaining pins functionality is confirmed and implemented.

![Curent Pinout](stm32f_pinout.png)

## Remaining Peripherals


### Monolithic Power MP2615
- Battery Charge Controller
- [MP2615 Datasheet](https://www.monolithicpower.com/en/documentview/productdocument/index/doc_url/L20vcC9tcDI2MTVfcjEuMDEucGRm/release_date/MjAxNC0wMS0yNiAwMDowMDowMA%3D%3D/)
- The MP2615 is a high efficiency switch mode
battery charger suitable for 1- or 2- cell lithium ion or lithium-Polymer applications. 

### Generic INA199
- Current Shunt Monitor (Battery Voltage)
- [INA199 Datasheet](http://www.ti.com/lit/ds/symlink/ina199.pdf)



### Disclaimer:
The firmware is distributed in the hope that it will be useful, but without any warranty. It is provided "as is" without warranty of any kind, either expressed or implied, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose. The entire risk as to the quality and performance of the firmware is with you.

All product and company names are trademarks™ or registered® trademarks of their respective holders. Use of them does not imply any affiliation with or endorsement by them.
