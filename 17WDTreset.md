[Index](Index.md)

# Introduction #

When the PIC turns on, determine how it was previously turned off. Print them out to the computer to see which one was responsible for the previous reset. Resetting the NU32 with the reset button, pulling the power, activating the software reset by reading a push button wired to E8, and letting the WDT run out.

Notice that the reset SPFs are cleared in software after they are read - they seem to stay at 1 forever, preventing you from knowing which one was the previous cause of reset.

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

  // check how the PIC was reset
  //   POR (power on reset - DNE for PIC with internal voltage reg like the 795)
  //   MCLR (reset button pushed)
  //   SWR (software reset - can trigger with SoftReset();)
  //   WDTR (watchdog timer reset)
  //   BOR (brown-out reset - voltage too low)
  //   CMR (config mismatch reset - code integrity check)
  int powerOnReset = RCONbits.POR;
  int externalReset = RCONbits.EXTR;
  int softwareReset = RCONbits.SWR;
  int wdtReset = RCONbits.WDTO;
  int brownOutReset = RCONbits.BOR;
  int configMismatchReset = RCONbits.CMR;

  // clear the possible reset values
  RCONbits.POR = 0;
  RCONbits.EXTR = 0;
  RCONbits.SWR = 0;
  RCONbits.WDTO = 0;
  RCONbits.BOR = 0;
  RCONbits.CMR = 0;

  sprintf(message,"POR: %d\r\n", powerOnReset);
  NU32_WriteUART1(message);

  sprintf(message,"EXTR: %d\r\n", externalReset);
  NU32_WriteUART1(message);

  sprintf(message,"SWR: %d\r\n", softwareReset);
  NU32_WriteUART1(message);

  sprintf(message,"WDTO: %d\r\n", wdtReset);
  NU32_WriteUART1(message);

  sprintf(message,"BOR: %d\r\n", brownOutReset);
  NU32_WriteUART1(message);

  sprintf(message,"CMR: %d\r\n", configMismatchReset);
  NU32_WriteUART1(message);

  // Print how many minutes have passed
  // Should not get to 18
  // service WDT and reset timer if USER is pushed
  // software reset if button on E8 is grounded
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
        if (!PORTEbits.RE8) {
          // software reset
          SoftReset();
        }
      }
      s++;
    }
  }

  return 0;
}

```