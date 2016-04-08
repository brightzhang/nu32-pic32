[Index](Index.md)

# Introduction #

Using the Comparator Reference Voltage internal voltage divider bridge, a 16-value analog output can be generated from the PIC32MX795F512L.

CVref can output 16 different analog values on CVrefOutput (B10). The values can be 0-2.06V when CVRR = 1 and 0.83-2.37V when CVRR = 0.

## Code ##
```
#include <plib.h>
#include "NU32.h"
#include <math.h>

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  // set up CVref
  CVRCONbits.CVRSS = 0; // use AVdd and AVss as reference
  CVRCONbits.CVRR = 1; // CVref voltages from 0-2.06V
  CVRCONbits.CVR = 0; // Make the reference voltage 0V
  CVRCONbits.CVROE = 1; // output CVref on B10
  CVRCONbits.ON = 1; // turn on CVref

  // make a single cycle sine wave with 100 values out of the 16 possible CVref values
  int i = 0;
  unsigned char wave[100];
  for (i=0;i<100;i++) {
    wave[i] = 8.0+8.0*sin(2.0*3.14*i/100.0);
  }

  // update the output at 1kHz to create a 10Hz sine wave
  WriteCoreTimer(0);
  while (1) {
    CVRCONbits.CVR = wave[i]; // set CVref

    // next value
    i++;
    if (i >= 100) {
      i = 0;
    }

    while(ReadCoreTimer() < 40000) {} // update at 1kHz for a 10Hz sine wave
    WriteCoreTimer(0);
  }

  return 0;
}
```