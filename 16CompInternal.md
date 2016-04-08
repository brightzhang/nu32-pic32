[Index](Index.md)

# Introduction #

In this sample code, the CMP is enabled to compare a voltage created by an external potentiometer circuit to an internally generated 1.03V. The output of the comparator is polled and used to control NU32LED1, and the output of the comparator is output to the comparator output pin to drive an LED as well.

Place the potentiometer output voltage on C1IN- (B4). Use the voltage divider bridge to set CVref of 1.03V. An LED on C1OUT (B8) is automatically controlled by the module, and NU32LED1 is controlled in software and should always have the same state as C1OUT (although note NU32LED1 is wired to be on when the output is false, so an if statement is used to make sure both LEDs are on at the same time).

When the potentiometer voltage (C1IN+) > 1.03V (C1IN-), C1OUT and NU32LED1 is high, otherwise low.

## Code ##
```
#include <plib.h>
#include "NU32.h"

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  // set up CVref
  CVRCONbits.CVRSS = 0; // use AVdd and AVss as reference
  CVRCONbits.CVRR = 0;
  CVRCONbits.CVR = 2; // Make the reference voltage 1.03V
  CVRCONbits.CVROE = 0; // don't output CVref on B10
  CVRCONbits.ON = 1; // turn on CVref

  // set up comparator 1
  CM1CONbits.COE = 1; // use C1OUT pin B8
  CM1CONbits.EVPOL = 0; // no interrupt
  CM1CONbits.CREF = 1; // use CVref as non-inverting input
  CM1CONbits.CCH = 0; // use C1IN- on B4 as inverting input
  CM1CONbits.ON = 1; // turn on comparator 1

  // if COUT is high, turn on NU32LED1
  // otherwise turn it off
  while (1) {
    if (CM1CONbits.COUT == 1) {
      NU32LED1 = 0; // on
    } else {
      NU32LED1 = 1; // off
    }
  }

  return 0;
}
```