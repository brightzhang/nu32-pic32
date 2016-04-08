# NU32 Sample code #

## Kit ##

[NU32 Kit](NU32Kit.md)

## 11. UART ##

  * Overview

UART (Universal Asynchronous Receiver Transmitter) is a full-duplex asynchronous serial peripheral.

Serial communication with UART is usually 8bit, with 1 start bit and 1 stop bit, and no parity bit (referred to as 8n1). Check the datasheet for the device you are trying to communicate with for the proper configuration.

The PIC uses TTL level signals - a low bit is 0V, and a high bit is 3.3V.

TTL is not compatible with 'RS-232' serial communication. RS-232, like that from a DB9 connector on a computer, contains bipolar signalling (+12V and -12V), and the signal is inverted. A logic level converter such as the MAX232 is required between TTL and RS-232 serial communication.

Each bit is transmitted at a baud (bits per second). The PIC has an adjustable baud, but there are several common bauds: 300 1200 2400 4800 9600 14400 19200 28800 38400 57600 115200 230400

Each byte is transmitted a a frequency of the baud/10.

The transmission is asynchronous - there is no shared clock signal between devices to signal when bits transition, as there is in I2C and SPI. The baud is agreed upon and assumed to be within a certain margin. If not, the data will be garbage.

The UART can use as few as two pins, RX (receive) and TX (transmit). The connection is RX to TX. Two additional pins can be used, RTS (request to send) and CTS (clear to send). RTS is an output that when high says to the other device, "I am ready for data, send away". When low, it is saying "I'm not prepared to receive your data, don't send it or it will be lost". This configuration is called Hardware Flow Control, and can be used to guarantee loss-less communication.

  * Details

The 795 has 6 UARTs.

The NU32 UART pins:

|x|RX|TX|RTS|CTS|
|:|:-|:-|:--|:--|
|U1|F2|F8|D15|D14|
|U2|F4|F5|F13|F12|
|U3|G7|G8|G6 |G9 |
|U4|D14|D15|x  |x  |
|U5|F12|F13|x  |x  |
|U6|G9|G6|x  |x  |

U1 and U3 are hardwired to the FT2232H chip to communicate with the computer, so they are not a good choice for communication with other devices. U4 and U6 share pins with U3 and U1 respectively, so really only U2 and U5 are usable on the NU32.

To use the peripheral, set the baud with BRGH in UxMODE and UxBRG = ((SYS\_FREQ / DESIREDBAUD) / 16) - 1;

Set the configuration with PDSEL, STEL, and UEN in UxMODE.

Enable the pins in UTXEN and URXEN in UxSTA.

Turn on the peripheral with ON in UxMODE.

When receiving data, check to see if there is anything in the buffer in UxSTA URXDA, and get the character from UxRXREG.

When sending data, check to see if there is room in the buffer in UxSTA UTXBF, and the set UxTXREG to the character to send. **wait for TRMT to be 1 to make sure it is sent before moving on?**

  * Library Functions

The NU32 uses U1 to communicate with the computer. Functions in NU32.h are helpful:

NU32\_WriteUART1(const char **string) calls**

WriteString(UART\_MODULE id, const char **string) calls**

PutCharacter(UART\_MODULE id, const char character)

NU32\_ReadUART1(char **message, int maxLength) will buffer charaters and block until it sees a '\n' or '\r'**

NU32\_EnableUART1Interrupt() will turn on the RX interrupt for U1 and

NU32\_DisableUART1Interrupt will turn it off

Very useful to understand the sprintf and sscanf functions, to create or read from a string or character array. %d for signed int, %u for unsigned int, %c for ascii charater, %f for float

  * Sample Code
    1. [loopback](11loopback.md), PIC talks to itself  **TESTED**
    1. [PIC to PIC](11PICtoPIC.md) **TESTED** _both NU32s need debugging ports open, maybe change over to toggling leds_
    1. [HFC PIC to PC with FTDI cable](11HardwareFlowControl.md) **TESTED**
    1. [XBee](11XBee.md) **TESTED** _could do more: pic changes xbee parameters, pic uses xbee as mobile uc, stress test xbee throughput_
    1. [PIC to MATLAB](11matlab.md) **TESTED**
    1. [PIC to Processing](11processing.md) **TESTED**
    1. [PIC to Chrome HTML5 app](11chrome.md) **TESTED**
  * Chapter Summary
  * Exercises

## 12. SPI ##
  * Overview
  * Details

SPI Pins:

|x|SCK|SDI|SDO|SS|
|:|:--|:--|:--|:-|
|SPI1|D10|C4 |D0 |D9|
|SPI2|G6 |G7 |G8 |G9|
|SPI3|D15|F2 |F8 |D14|
|SPI4|F13|F4 |F5 |F12|

SPI2 and SPI3 share pins with UART3 and UART1 respectively, so they cannot be used on the NU32.

  * Library Functions
  * Sample Code
    1. [loopback](12SPIloopback.md), PIC talks to itself **UNTESTED**
    1. [PIC to PIC](12SPIPICtoPIC.md) **UNTESTED**
    1. [DAC](12SPIDAC.md) **TESTED**
    1. [ADC](12SPIADC.md) **TESTED**
    1. [SRAM](12SPISRAM.md) **UNTESTED**
    1. [uSD card](12SPIuSD.md) **UNTESTED**
    1. [ADXL362 accelerometer](12SPIadxl362.md) **TESTED**
  * Chapter Summary
  * Exercises

## 13. I2C ##
  * Overview
  * Details

I2C pins:

|x|SDA|SCL|
|:|:--|:--|
|I2C1|A15|A14|
|I2C2|A3 |A2 |
|I2C3|F2 |F8 |
|I2C4|G7 |G8 |
|I2C5|F4 |F5 |

I2C3 and I2C4 share pins with UART1 and UART3 respectively, so they cannot be used on the NU32.

  * Library Functions
  * Sample Code
    1. [loopback](13I2Cloopback.md), PIC talks to itself **UNTESTED**
    1. [PIC to PIC](13I2CPICtoPIC.md) **UNTESTED**
    1. [DAC](13I2CDAC.md) **UNTESTED**
    1. [ADC](13I2CADC.md) **UNTESTED**
    1. [SRAM](13I2CSRAM.md) **UNTESTED**
    1. [accelerometer/compass](13I2CAccelCompass.md) **UNTESTED**
  * Chapter Summary
  * Exercises

## 14. USB ##
  * Overview
  * Details
  * Library Functions
  * Sample Code
    1. [HID](14USBHID.md), general, mouse, keyboard **UNTESTED**
    1. [CDC](14USBCDC.md) **UNTESTED**
    1. [Thumb drive](14USBMassStorage.md) **UNTESTED**
    1. [Android](14USBAndroid.md) **UNTESTED**
    1. [Bluetooth, Wifi dongle?](14USBdongle.md) **UNTESTED**
  * Chapter Summary
  * Exercises

## 15. Input Capture ##
  * Overview

This peripheral captures a timer value on rising or falling edges, allowing for pulse width and frequency measurements.

  * Details

The 795 has five IC pins, IC1 on D8, IC2 on D9, IC3 on D10, IC4 on D11, IC5 on D12.

Uses Timer 2 and/or Timer 3. Captured events are stored in a four-level deep buffer. The ICBNE can be polled to see if there is anything in the buffer, or an interrupt can be generated when a capture occurs.

  * Library Functions
  * Sample Code
    1. [Loopback with PWM](15ICloopback.md) **TESTED**
    1. [Read from Hobbyking RX](15ICinput.md) **TESTED**
  * Chapter Summary
  * Exercises

## 16. Comparator ##
  * Overview

This peripheral contains two comparator modules. The two inputs can come from 2 external sources, an internal voltage reference (CVref) with 16 values, or an internal absolute voltage reference of 0.6V or 1.2V. The output can be read in code, generate an interrupt, or output directly to a pin. CVref can be output on a pin as well, creating an analog output.

  * Details

CVref can take 16 values 0.83-2.37V or 0-2.06V depending on CVRR, with an output impedance of ~15k.

On the 795, C1IN+ is B5, C1IN- is B4, C2IN+ is B3, C2IN- is B2. C1OUT is B8, C2OUT is B9. CVREFout is B10.

  * Library Functions
  * Sample Code
    1. [Compare to internally set voltage](16CompInternal.md) **TESTED**
    1. [16 value output](16CompOutput.md) **TESTED**
    1. [compare two inputs, interrupt](16CompStandard.md) **TESTED**
  * Chapter Summary
  * Exercises

## 17. Watchdog timer and Resets ##
  * Overview

The WDT uses the low-power RC oscillator clock source. Timeouts from 1 ms to 1048 s can be used. The WDT is "serviced" by setting WTDCLR in WTDCON.

When the PIC turns on, you can check to see if the previous reason for turning off was from POR (power on reset - DNE for PIC with internal voltage reg like the 795), MCLR (reset button pushed), SWR (software reset - can trigger with RSWRSTSET = 1; while(1);), WDTR (watchdog timer reset), BOR (brown-out reset - voltage too low), or CMR (config mismatch reset - code integrity check).

  * Details

If the configuration bit FWDTEN is set, the WDT is always enabled, and ON is always set and cannot be cleared. If the config bit is not set, the WDT can be turned on and off in software.

There are two modes, Windowed, where the WDT can be reset at any time, and Programmable Windowed, where the WDT can only be reset in the final window, otherwise a reset will occur (not sure why you would ever want this).

  * Library Functions
  * Sample Code
    1. [enable and reset](17WDTstandard.md) **TESTED**
    1. [check reason for previous reset](17WDTreset.md) **TESTED**
  * Chapter Summary
  * Exercises

## 18. Flash storage ##
  * Overview
  * Details
  * Sample Code using PLIB
    1. [write some data, reset and read it back](18Flash.md) **TESTED**
  * Chapter Summary
  * Exercises

## 19. Power saving ##
  * Overview

The PIC has nine low-power modes to balance power use with performance. Four of these mode involve switching SYSCLK and PBCK to lower frequencies by other sources, and five of these modes can halt (idle) the CPU but the peripherals continue to run on SYSCLK.

  * Details
  * Library Functions
  * Sample Code
    1. [Sleep](19PSsleep.md) **TESTED**
  * Chapter Summary
  * Exercises

## 20. RTCC ##
  * Overview

This peripheral can keep track of the day, date and time to half a second. An alarm can be set and output to the RTCC pin on D8, or each half second tick can be output on the RTCC pin. Time is saved as binary-coded decimal value (BCD).

  * Details

Requires a 32.768 kHz crystal on the secondary oscillator, SOSCO on pin C14 and SOSCI on pin C13, with appropriate load capacitors, generarlly 10-30pF (C13 is the USER pin on the NU32, we should move the USER pin).

  * Library Functions
  * Sample Code
    1. [Set the time and read it back](20RTCC.md) **TESTED**
  * Chapter Summary
  * Exercises

## 21. CAN ##
  * Overview
The controller area network bus (CAN) is an asynchronous, message-based communication protocol, originally developed for the automotive industry. The CAN peripheral on the PIC32 manages CAN transmission, reception, and error handling of the CAN messages, with minimal effort on the front end by the programmer. The CAN peripheral requires a physical layer - a transceiver interface for the two-wire differential CAN bus. We will use the MCP2562, available as a DIP8, supporting logic-level IO and 1Mbit/s.
  * Details
The 795 has two can modules, C1RX on F0 and C1TX on F1, and C2RX on G0 and C2TX on G1. Alternate pins are available, AC1RX on F12 and AC1TX on F13, and AC2RX on C3 and AC2TX on C2.

A CAN message is made of dominant bits (zeros) and recessive bits (ones). Messages are broadcast synchronously. The first part of the message contains the identifier bits. Each device on the CAN bus transmitting a message with simultaneously submit their identifier, but will stop when they have submitted a recessive bit while another has submitted a dominant bit (arbitration). If a device has lost arbitration, it will try to submit again on the next message. In this way, the lower message IDs take priority. After gaining arbitration, a device will continue to submit bits, including data and CRC. All devices listening to the bus will calculate their own CRC and compare, with several methods of detecting errors and preventing damaged devices from hogging the bandwidth.

The CAN peripheral on the PIC32 has direct access to the system bus, and takes care of arbitration and synchronizing with the rest of the CAN bus. In this way, the peripheral can directy access RAM when a message needs to be sent or is received. RAM must be reserved for the messages. The peripheral is very flexible to account for up to 32 FIFO messages of 2 or 4 words each (2 for RX listen only, usually use 4).

A CAN FIFO is declared as TX or RX (cannot be both). For TX, RAM is reserved and the physical address is pointed to by CxFIFOBA. Each FIFO to be used is given a size and TX is enabled while the peripheral is in Config mode. The peripheral is moved to Normal mode, ID and data is put into the RAM, the UINC bit is set to automatically increment the point to the next message, and the TXREQ bit is set to trigger and transmission. For RX, The same process is used, except TXEN is not set, and filters are set up to compare with message ID. If a message has an ID that matches the filter, the message is stored to RAM and a flag is set. On interrupt or polling, the data is pulled out of the message.

RAM is not preformatted for the message structure, so a typdef struct and union in the plib CAN functions is used. The union allows for easy clearing of the message, while also accessing the individual data and ID bytes in the message. The format is very flexible to allow for different length IDs and size of messages.

  * Library Functions
  * Sample Code
    1. [PIC to PIC](21CANPICtoPIC.md) **TESTED**
  * Chapter Summary
  * Exercises

## 22. DMA ##
  * Overview
  * Details
  * Library Functions
  * Sample Code
    1. [ADC](22DMAADC.md) **UNTESTED**
    1. [SPI](22DMASPI.md) **UNTESTED**
  * Chapter Summary
  * Exercises