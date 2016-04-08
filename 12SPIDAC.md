[Index](Index.md)

# Introduction #

SPI to MCP4922 DAC

The MCP49922 DAC is a dual 12 bit DAC.

Pin 1 is VDD to 3.3V, pin 3 is chip select (to PIC F12 SS4) pin 4 is SCK (to PIC F13 SCK4), pin 5 is SDI (to PIC F5 SD)4), pin 8 is LDAC to GND, pin 9 is SHDN to 3.3V, pin 10 is channel B out, pin 11 is VREFB to 3.3V, pin 12 is GND to GND, pin 13 is VREFA to 3.3V, and pin 14 is channel A out.

LDAC is connected to GND so that the chip will update when CS goes high.

The communication is 16 bit. Bit 15 is the channel, bit 14 is if the ref is buffered, bit 13 is the gain, bit 12 is shutdown, and bit 11-0 are the data.

The DAC does not respond, so SDI4 on F4 is not used.

Using SPI4 as master with slave select, update channel A and B of the dac at 10kHz to produce a 100Hz sine and triangle wave.


### Code ###

```
#include <plib.h>
#include "NU32.h"
#include <math.h>

void writeToMCP4922(int channel, int gain, int value);

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  // init SPI4
  SPI4CON = 0; // Disable and reset SPI module
  SPI4BRG = 3; // Set the baudrate to (80MHz)/2/(3+1) = 10Mhz
  SPI4STATbits.SPIROV = 0; // Clear overflow flag

  SPI4CONbits.MSTEN = 1; // Enable Master Mode
  SPI4CONbits.MODE16 = 1; // Data is 16 bits wide
  SPI4CONbits.CKP = 0; // Clock is active high
  SPI4CONbits.CKE = 1; // Serial Data changes on active to idle
  SPI4CONbits.SSEN = 1; // use the SS pin
  SPI4CONbits.MSSEN = 1; // use the SS pin functionality

  SPI4CONbits.ON = 1; // Turn the SPI module on

  // make a sine wave and a triangle wave to output
  int i;
  int sineWave[100];
  int triWave[100];
  for(i=0;i<100;i++) {
    sineWave[i] = 4096/2+4096/2*sin(2.0*3.14*i/100.0);

    if(i<50) {
      triWave[i] = i*4095/50;
    }
    else {
      triWave[i] = 4095-(i-50)*4095/50;
    }
  }

  // output the waves at 100Hz (10kH update rate)
  i = 0;
  while (1) {
    WriteCoreTimer(0);

    writeToMCP4922(0, 1, sineWave[i]); // channel A, gain of 1
    writeToMCP4922(1, 1, triWave[i]); // channel B, gain of 1
    i++;

    if(i == 100) {
      i = 0;
    }

    while(ReadCoreTimer() < 4000) {}
  }
  return 0;
}

void writeToMCP4922(int channel, int gain, int value)
{
    // DAC output, Vout = gain*Vref*value/4096;
    int output = 0;
    int input = 0;

    output |= (channel << 15);  // Set the channel
    output |= (gain << 13);     // Set the gain
    output |= (1 << 12);        // Set output to be active (0 for inactive)
    output |= (value & 0x0FFF);  // Set the last 12 bits of value to be

    SPI4BUF = output;
    while(!SPI4STATbits.SPIRBF);  // Wait for transaction to finish
    input = SPI4BUF;        // Read the buffer to clear it

    return;
}
```