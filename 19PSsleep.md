[Index](Index.md)

# Introduction #

In idle mode, the CPU is halted when an 'asm volatile("wait")' command is issued. The CPU will turn back on when an interrupt with priority higher than zero occurs.

In this code the LEDs blink, but stop when the USER button puts the PIC into idle mode. Change notification on C14 generates an interrupt to wake the PIC up and the code continues to blink the LEDs.

## Code ##
```

#include <plib.h>
#include "NU32.h"

int main(void) {
  NU32_Startup();
  NU32LED1 = 0; // on
  NU32LED2 = 1; // off

  // enable oscillator sleep
  // will go to sleep with a wait asm
  OSCCONbits.SLPEN = 1;

  // enable change notification on C14 CN0 with pullup
  CNCONbits.ON = 1;
  CNPUEbits.CNPUE0 = 1;
  CNENbits.CNEN0 = 1;
  PORTC; // read C to prevent immediate interrupt

  // turn on CN interrupt
  IPC6bits.CNIP = 5; // interrupt priority 5
  IPC6bits.CNIS = 0;
  IFS1bits.CNIF = 0; // clear the flag
  IEC1bits.CNIE = 1; // turn on the interrupt

  // toggle the LEDs at 1Hz
  // if USER is pushed, go to sleep, LEDs should stop blinking
  // if C14 (CN0) is grounded, wake up and start blinking
  while (1) {
    // toggle the LEDs
    NU32LED1 = !NU32LED1;
    NU32LED2 = !NU32LED2;

    WriteCoreTimer(0);
    while(ReadCoreTimer()<40000000) {
      if (!NU32USER) {
        asm volatile("wait");
      }
    }
  }

  return 0;
}

// wake up the PIC
void __ISR(_CHANGE_NOTICE_VECTOR, ipl5) CNHandler(void) {
  // read C14
  PORTC;

  // Clear the interrupt Flag
  IFS1bits.CNIF = 0;
}

```