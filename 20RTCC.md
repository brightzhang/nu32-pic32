[Index](Index.md)

# Introduction #

In this sample code, you can set the date, day and time of the RTCC. A square wave will output a on the RTCC pin (D8) with a 1 second period.

There must be a 32.768 kHz crystal on the secondary oscillator, SOSCO on pin C14 and SOSCI on pin C13, with appropriate load capacitors, generally 10-30pF.

Some changes to the (2013 and 2014 NU32 board) and (all versions of the) bootloader are necessary, so this sample code does not use the bootloader or the 10k pullup resistor on USER.

MAYBE PUT THE CRYSTAL ON THE 2015 BOARD AND ADD THE ABILITY TO USE IT IN THE BOOTLOADER?

## Code ##
```

#include <plib.h>
#include "NU32.h"

#define NU32_STANDALONE

#ifdef NU32_STANDALONE              // config bits if not set by bootloader

#pragma config DEBUG = OFF          // Background Debugger disabled
#pragma config FPLLMUL = MUL_20     // PLL Multiplier: Multiply by 20
#pragma config FPLLIDIV = DIV_2     // PLL Input Divider:  Divide by 2
#pragma config FPLLODIV = DIV_1     // PLL Output Divider: Divide by 1
#pragma config FWDTEN = OFF         // WD timer: OFF
#pragma config POSCMOD = HS         // Primary Oscillator Mode: High Speed xtal
#pragma config FNOSC = PRIPLL       // Oscillator Selection: Primary oscillator w/ PLL
#pragma config FPBDIV = DIV_1       // Peripheral Bus Clock: Divide by 1
#pragma config BWP = OFF            // Boot write protect: OFF
#pragma config ICESEL = ICS_PGx2    // ICE pins configured on PGx2, Boot write protect OFF.

/* CHANGED TO ON TO USE C13 AND C14 ALSO REMOVED USER PULLUP*/
#pragma config FSOSCEN = ON        // Enable second osc

#endif // NU32_STANDALONE

char * weekdays[7] = {"sunday", "monday", "tuesday", "wednesday", "thursday", "friday", "saturday"};

int main(void) {
  NU32_Startup();
  NU32LED1 = 0; // on
  NU32LED2 = 1; // off

  char message[100];

  // these don't seem to be necessary
  OSCCONbits.SOSCEN = 1; // enable the secondary osc, pins C13 and C14
  while(!OSCCONbits.SOSCRDY);

  // unlock to allow changes to the RTCC
  SYSKEY = 0xaa996655; // write first unlock key to SYSKEY
  SYSKEY = 0x556699aa; // write second unlock key to SYSKEY
  RTCCONbits.RTCWREN = 1; // allow changes to the date and time

  RTCCONbits.ON = 0; // turn off the RTCC

  while(RTCCONbits.RTCCLKON); // wait for clock to turn off

  // set the time to 23:58:45am (using the bits doesn't seem reliable)
  unsigned long time = 0x23584500;
  RTCTIME = time;

  // set the date to Wed July(7) 24 2013 (for WDAY01, Sun = 0)
  RTCDATEbits.YEAR10 = 1;
  RTCDATEbits.YEAR01 = 3;
  RTCDATEbits.MONTH10 = 0;
  RTCDATEbits.MONTH01 = 7;
  RTCDATEbits.DAY10 = 2;
  RTCDATEbits.DAY01 = 4;
  RTCDATEbits.WDAY01 = 3;

  RTCCONbits.RTSECSEL = 1; // output the second square wave on the RTCC pin
  RTCCONbits.RTCOE = 1; // output on RTCC pin D8
  RTCCONbits.ON = 1; // turn on the RTCC

  RTCCONbits.RTCWREN = 0; // prevent changes to the time

  while(!RTCCONbits.RTCCLKON); // wait for clock to turn on

  // write the time and date back to verify
  sprintf(message, "The date is %s %d%d/%d%d/20%d%d \r\n", weekdays[RTCDATEbits.WDAY01], RTCDATEbits.MONTH10, RTCDATEbits.MONTH01, RTCDATEbits.DAY10, RTCDATEbits.DAY01, RTCDATEbits.YEAR10, RTCDATEbits.YEAR01);
  NU32_WriteUART1(message);

  sprintf(message, "The time is %d%d:%d%d:%d%d \r\n", RTCTIMEbits.HR10, RTCTIMEbits.HR01, RTCTIMEbits.MIN10, RTCTIMEbits.MIN01, RTCTIMEbits.SEC10, RTCTIMEbits.SEC01);
  NU32_WriteUART1(message);

  // write the date and time every two seconds
  while (1) {
    sprintf(message, "The date is %s %d%d/%d%d/20%d%d \r\n", weekdays[RTCDATEbits.WDAY01], RTCDATEbits.MONTH10, RTCDATEbits.MONTH01, RTCDATEbits.DAY10, RTCDATEbits.DAY01, RTCDATEbits.YEAR10, RTCDATEbits.YEAR01);
    NU32_WriteUART1(message);

    sprintf(message, "The time is %d%d:%d%d:%d%d \r\n", RTCTIMEbits.HR10, RTCTIMEbits.HR01, RTCTIMEbits.MIN10, RTCTIMEbits.MIN01, RTCTIMEbits.SEC10, RTCTIMEbits.SEC01);
    NU32_WriteUART1(message);

    // wait 2 seconds
    WriteCoreTimer(0);
    while (ReadCoreTimer() < 80000000) {
    }
  }

  return 0;
}

```