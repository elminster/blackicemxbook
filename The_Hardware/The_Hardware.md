# The Hardware

##	The MyStorm BlackIce Mx Development Board

![myStorm BlackIce Mx][img1]

The [myStorm BlackIce Mx][] is a development board with:
*	A Lattice HX4K TQFP144 Ice40 FPGA with 16KB of block RAM, running at 216MHz
*	An STM32F730 processor with 256KB of flash memory and 64KB or RAM
*	A  USB 2.0 full-speed port attached to the STM32 used for programming and as a UART to the ice40
* A second USB port attached to the STM32, currently unused
*	3 Mixmods which correspond to 6 double Pmods but also have analog signalks
* 2MB (!6 MBbit) of external SDRAM
* 512KB (4 Mbit) of external flash memory 
*	An SD card reader available to the ice40
*	A short Raspberry Pi header
*	Two buttons available to the FPGA
*	A power LED
*	A red (D) LED that indicates when the FPGA has been configured (CDONE)
*	A green (S) LED available to the STM32 (STATUS)
* A yellow (M) LED that indicates the programming mode (MODE)
*	4 user LEDs (B, G, Y, R) available to the FPGA, but two shared with the buttons
* A mini HDMI connector (not currently working)

The standard [mystorm][] STM32 firmware supports configuring the FPGA by copying the bit stream to the CDC-ACM USB port (/dev/ttyACM0 on Linux). It also supports writing bitstreams to the flash memory, which the ice40 then boots to.

It is possible to write custom firmware that can provide extra functions but must provide some way to configure the FPGA.

As the icestorm tools treat the HX4K device as an HX8K device, the specs of the HX8K are most relevant to BlackIce Mx.

[img1]:					https://cdn.tindiemedia.com/images/resize/kRnwcslzIExDv1SM6Nmc1doFpqI=/p/fit-in/774x516/filters:fill(fff)/i/7474/products/2019-07-03T14%3A53%3A37.320Z-BlackIceMx.jpg
[myStorm BlackIce Mx]:	https://www.tindie.com/products/Folknology/blackice-mx/
[mystorm]:				https://github.com/folknology/IceCore/tree/USB-CDC-issue-3/firmware/myStorm

##	The Lattice ICE40 HX4K FPGA
As the device is effectively an HX8K when programmed with the icestorm tools, the specs for that device are most relevant. It comes in a TQFP144 144 pin package.  There are 48 pins available via the Pmod connectors.

There are two PLLs and 960 Programmable Logic Blocks (PLBs) which are each a block of 16 logic cells (LCs), giving a total of 7680 LCs. Each logic cell consists of a 4 input look-up table (4-LUT) and a 1-bit D flip-flop. There are 32 BRAM blocks, each of 512 bytes, giving a total of 16KB of BRAM.

Verilog written for other devices, provided it does not use directives specific to Xilinx or Altera or other manufacturers, will normally work on the Ice40, as long as it does not use too many resources. The icestorm tools report on all resources used.

There are some differences due to use of the icestorm tools. It is not usually possible to deduce the usage of pins marked as inout in module interfaces, and there is limited support for tri-state logic. For this reason, the SB_IO directive (see the Directives chapter below) needs to be used for inout pins and the should be tri-stated by setting the direction to input.

Although all the examples in this book are in Verilog, it is possible to program the device using higher-level hardware description languages (HDLs). See the Languages chapter.

##	STM32 Processor
The STM32F7303 is a 32-bit ARM device that shares some pins with the Ice40 and with the Rpi header.

The main ways of programming the STM32 processor is to use the arm-none-eabi gnu toolchain with STM HAL libraries. This is usually done by copying and modifying the myStorm firmware.  This is described in the STM32 Programming chapter below.

##	Pins

![pinout][img2]

[img2]: ./pinout.png "pinout"

### The Clock

Pin 60 is the external clock, connected to a 25 Mhz oscillator.  This clock, or clocks derived from it, are used to co-ordinate all sequential logic.

Clocks of other frequencies can be derived from this clock pin by use of clock dividers or Phase Lock Loops (PLLs). PLLS are described in the Directives chapter.


### User buttons

Button1 and button2 correspond to Ice40 pins 49 and 52, They are pulled up to 3.3v by a 10k resistor, so they are pulled low when pressed. Pin 49 is connected to the blue LED and pin 52 to the green LED, so those LEDs come on when the corresponding button is pressed. Although these pins have a pull-up resistor, the ice40 internal pull-up resistor also needs to be enabled for reliable operations of the buttons.

### LEDs

The LEDS correspond to pins 49, 52, 55 and 56. Pins 49 and 52 are also buttons.

## USB Port 

The USB port, USB PRG, is connected to the STM32 processor which can support full USB capability. The iceboot software uses it as a CDC-ACM serial communication port that runs at full speed and is available as /dev/ttyACM0 on Linux. You do not need to set a baud rate for this port as it runs at full speed, but you should set it to raw and no echo by `stty -F /dev/ttyACM0 raw -echo` for the ice40 bitstream configuration to work correctly in binary (raw) mode.

Custom firmware on the STM32 can use both USB ports in any way that it wants.

## RPi header

The Rpi header allows the Ice40 and the STM32 to talk to a Raspberry Pi using a ribbon cable or directly to a Raspberry Pi Zero plugged in to the headed via a female header on the underside of the Pi Zero.

| WiringPi | GPIO   | Usage  | Phys | Pin# | Usage | GPIO   | WiringPi |
| -------- |:----   |:-----  |:---- |:---- |:----- |:----   |:-------- |
|          |        |  NC    |  1   |  2   |  5V   |        |          |
|    8     | GPIO2  |  SDA   |  3   |  4   |  5V   |        |          |
|    9     | GPIO3  |  SCL   |  5   |  6   |  GND  |        |          |
|    7     | GPIO4  | SWCLK  |  7   |  8   |  RX   | GPIO14 |    15    |
|          |        |  GND   |  9   |  10  |  TX   | GPIO15 |    16    |
|    0     | GPIO17 | SWDIO  |  11  |  12  |  TCK  | GPIO18 |    1     |
|    2     | GPIO27 | RESET  |  13  |  14  |  GND  |        |          |
|    3     | GPIO22 |  TMS   |  15  |  16  |  TDI  | GPIO23 |    4     |
|          |        |   NC   |  17  |  18  |  TDO  | GPIO24 |    5     |
|    12    | GPIO10 |  EMOSI |  19  |  20  |  GND  |        |          |
|    13    | GPIO9  |  EMISO |  21  |  22  | DONE  | GPIO25 |    6     |
|    14    | GPIO11 |  ESCK  |  23  |  24  |  ESS  | GPIO8  |    10    |
|          |        |  GND   |  25  |  26  |  NC   | GPIO7  |    11    |


## Schematics

### IceCore

![IceCore Schematic][img3]

### Blackice Mx

![BlackIce Mx Schematic][img4]

[img3]:				./IceCore-schematic.png "IceCore Schematic"
[img4]:				./BlackIceMx-schematic.png "BlackIce Mx Schematic"

|                        |                        |                        |
|------------------------|------------------------|------------------------|
|[Previous](../Introduction/Introduction.html)|[Up](..) |[Next](../Programming_the_Built-in_Hardware/Programming_the_Built-in_Hardware.html)|
