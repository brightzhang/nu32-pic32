[Index](Index.md)

# Introduction #

Instead of polling the output of the comparator, we can use the output to generate an interrupt. In this case, we will have the comparator output directly control an LED, and have the interrupt control another LED to always have the same state.

Two externally generated analog voltages are compared.

Compare a voltage divider voltage (made from 10k resistors, 1.65V) on C1IN+ (B5) to a potentiometer output on C1IN- (B4). An LED on C1OUT (B8) is automatically controlled by the module, and NU32LED1 is controlled in software to match.

When C1IN+ > C1IN-, C1OUT is high, otherwise low. An interrupt controls NU32LED1 when C1OUT changes.

## Code ##
```
#include <plib.h>
#include "NU32.h"

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  CM1CONbits.COE = 1; // use C1OUT pin B8
  CM1CONbits.EVPOL = 3; // interrupt on L-H and H-L C1OUT transitions
  CM1CONbits.CREF = 0; // use C1IN+ on B5
  CM1CONbits.CCH = 0; // use C1IN- on B4
  CM1CONbits.ON = 1; // turn on comparator 1

  // Enable interrupt for comparator 1
  IPC7bits.CMP1IP = 4;
  IPC7bits.CMP1IS = 1;
  IFS1bits.CMP1IF = 0;
  IEC1bits.CMP1IE = 1;

  while (1) {
    // nothing, use the interrupt
  }
  return 0;
}

void __ISR(_COMPARATOR_1_VECTOR, ipl4) Cmp1_IntHandler (void) {
  if (CM1CONbits.COUT == 1) {
    NU32LED1 = 0; // on
  }
  else {
    NU32LED1 = 1; // off
  }

  IFS1bits.CMP1IF = 0; // clear the interrupt flag
}
```