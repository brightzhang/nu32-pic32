[Index](Index.md)

# Introduction #

Enable the WDT and reset it if the NU32USER button is pushed. Print out how much time has passed to the computer. Without resetting the WDT, the PIC will automatically reset after 1048 seconds (17 minutes) so the count time passed should roll back to 0 after 17 minutes.

## Code ##

```

#include <plib.h>
#include "NU32.h"

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  char message[100];
  int s = 0, m = 0;

  // setup the watchdog timer
  // programmable windowed mode (no WDTCONbits.WDTWINEN, bit 1?)
  // by default, #pragma config WDTPS 0b10100, so reset in 1048.576s (17 min 28.57 s)
  WDTCONbits.ON = 1; // turn on the WDT

  sprintf(message, "Code started:\r\n");
  NU32_WriteUART1(message);

  // Print how many minutes have passed
  // Should not get to 18
  // service WDT and reset timer if USER is pushed
  while (1) {
    sprintf(message, "%d minutes have passed\r\n", m);
    NU32_WriteUART1(message);
    m++;
    s = 0;

    // count up to 60 seconds
    while (s < 60) {
      // wait 1 second while checking USER
      WriteCoreTimer(0);
      while (ReadCoreTimer() < 40000000) {
        if (!NU32USER) {
          WDTCONbits.WDTCLR = 1;
          sprintf(message, "Watchdog reset!\r\n");
          NU32_WriteUART1(message);
          s = 0;
          m = 0;
        }
      }
      s++;
    }
  }

  return 0;
}

```