[Index](Index.md)

# Introduction #

SPI to MCP3008 ADC

The MCP3008 is an 8 channel single-ended, or 4 channel differential ADC. Conversion uses the SPI clock.

In this code, SPI4 with automatic slave select is used to read channel 0 and channel 1 in single-ended mode at 100Hz. The data is sent to the computer to display.


## Code ##

```
// Communicate with the MCP3008 8 channel, 10 bit ADC over SPI
// read channels 0 and 1, send the data to the computer to plot

// channel 0 is pin 1, channel 1 is pin 2, both have a potentiometer from 0-3.3V
// pin 8 and 14 anf gnd, pin 15 and 16 are 3.3V
// pin 9 is SS4 on F12
// pin 10 is data in, to SDO4 on F5
// pin 11 is data out, to SDI4 on F4
// pin 12 is SCK to F13

#include <plib.h>
#include "NU32.h"

int readFromMCP3008(int channel);

int main(void) {
  NU32_Startup();

  int ADCValue0, ADCValue1;
  char messageArray[100];

  // set up SPI4
  SPI4CON = 0; // Disable and reset SPI module
  SPI4BRG = 39; // Set the baudrate to (80MHz)/2/(39+1) = 1Mhz
  SPI4STATbits.SPIROV = 0; // Clear overflow flag
  SPI4CONbits.MSTEN = 1; // Enable Master Mode
  SPI4CONbits.MODE32 = 1; // Data is 32 bits wide
  SPI4CONbits.CKP = 0; // Clock is active high
  SPI4CONbits.CKE = 1; // Serial Data changes on active to idle
  SPI4CONbits.SSEN = 1; // use the SS pin
  SPI4CONbits.MSSEN = 1; // use the SS pin functionality
  SPI4CONbits.ON = 1; // Turn the SPI module on

  while(1) {
    WriteCoreTimer(0);
    ADCValue0 = readFromMCP3008(0);
    ADCValue1 = readFromMCP3008(1);
    sprintf(messageArray,"%d %d\n",ADCValue0/2,ADCValue1/2);
    NU32_WriteUART1(messageArray);

    // delay for output at 100Hz
    while(ReadCoreTimer()<400000) {}
  }
}

int readFromMCP3008(int channel) {
  int output = 0;
  int input = 0;
  int ADCValue = 0;

  // ADC input = 1024*Vin/Vref
  output |= (1 << 16); // Start bit
  output |= (1 << 15); // Differential = 0; Single-ended input = 1;
  output |= ((channel & 7) << 12); // Set the channel(max 7)

  SPI4BUF = output;
  while(!SPI4STATbits.SPIRBF); // Wait for transaction to finish
  input = SPI4BUF; // Read the buffer to clear it
  ADCValue = input & 0x3FF; // Ignore all but last 10 bits

  return ADCValue;
}

```