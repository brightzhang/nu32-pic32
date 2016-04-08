[Index](Index.md)

# Introduction #

Most remote control (RC) vehicles are controlled by 2.4GHz transceivers. The angles of joysticks are sent wirelessly to control motor speeds or servo angles. Because servo angles are controlled by very specific pulse widths, the output of the receiver is a pulse width so the servo can be plugged into the receiver directly.

A 2.4GHz transceiver can transmit control data several hundred yards, and are available as low as $25, such as the [3 channel Hobby King](http://www.hobbyking.com/hobbyking/store/__31671__HobbyKing_HK_GT2B_3CH_2_4GHz_Transmitter_and_Receiver_w_Rechargable_Li_ion_Battery.html). The specifications say that 3 channels of 10-bit data are sent.

The receiver runs on 5V. The output signals repeat at 18.5mS with pulses 1mS to 2mS in duration.

Using Input Capture 1 (D8), 2 (D9) and 3 (D10) to acquire the pulse durations from the RX.

Use Timer 3 with a prescalar of 8 as a reference (0.1uS resolution). There will be timer rollover between pulses, so correct for that in the code.

Send the pulse duration as a value from 0 to 512 at 40Hz to the NU32Utility plotting tool to view the data as a graph in real time.

## Code ##
```
#include <plib.h>
#include "NU32.h"
#include <math.h>

int pulse1 = 0, pulse2 = 0, pulse3 = 0;

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  char message[100];

  // set up Input Capture 1 on D8, 2 on D9, 3 on D10, with T3
  T3CONbits.TCKPS = 3; // set prescaler 1:8
  TMR3 = 0; // start T3 at 0
  // each pulse is 1 to 2mS, unknown resolution but 10000 seems plenty
  PR3 = 65535; // max period match = 6.6mS, each count is 8/80MHz = 0.1uS

  // IC1
  IC1CONbits.C32 = 0; // use 16 bit
  IC1CONbits.ICTMR = 0; // use T3
  IC1CONbits.FEDGE = 1; // capture rising edge first
  IC1CONbits.ICM = 6; // capture every edge starting with the above
  IC1CONbits.ICI = 1; // interrupt on every 2nd capture event
  IPC1bits.IC1IP = 3;
  IPC1bits.IC1IS = 0;
  IEC0bits.IC1IE = 1; // enable IC1 interrupt

  // IC2
  IC2CONbits.C32 = 0; // use 16 bit
  IC2CONbits.ICTMR = 0; // use T3
  IC2CONbits.FEDGE = 1; // capture rising edge first
  IC2CONbits.ICM = 6; // capture every edge starting with the above
  IC2CONbits.ICI = 1; // interrupt on every 2nd capture event
  IPC2bits.IC2IP = 3;
  IPC2bits.IC2IS = 0;
  IEC0bits.IC2IE = 1; // enable IC2 interrupt

  // IC3
  IC3CONbits.C32 = 0; // use 16 bit
  IC3CONbits.ICTMR = 0; // use T3
  IC3CONbits.FEDGE = 1; // capture rising edge first
  IC3CONbits.ICM = 6; // capture every edge starting with the above
  IC3CONbits.ICI = 1; // interrupt on every 2nd capture event
  IPC3bits.IC3IP = 3;
  IPC3bits.IC3IS = 0;
  IEC0bits.IC3IE = 1; // enable IC3 interrupt

  IC1CONbits.ON = 1; // turn on IC1
  IC2CONbits.ON = 1; // turn on IC2
  IC3CONbits.ON = 1; // turn on IC3
  T3CONbits.ON = 1; // turn on T3

  // send pulse values to computer at 40Hz
  WriteCoreTimer(0);
  while (1) {
    // pulse values are in 1uS counts, expected in the range 10000-20000
    // subtract 10000 and divide by 20 to get in the range for plotting in NU32Utility
    sprintf(message,"%d %d %d\r\n", (pulse1-10000)/20, (pulse2-10000)/20, (pulse3-10000)/20);
    NU32_WriteUART1(message);

    while(ReadCoreTimer()<40000000/40){}
    WriteCoreTimer(0);
  }

  return 0;
}

void __ISR(_INPUT_CAPTURE_1_VECTOR, ipl3) IC1Handler(void) {
  // Is this an IC1 interrupt?
  if (IFS0bits.IC1IF) {
    int a, b;
    // read data four times to clear interrupt
    a = IC1BUF; // oldest value
    b = IC1BUF; // newest value

    // check for timer overflow
    if (b > a) {
      pulse1 = b - a;
    }
    else {
      pulse1 = b + 65535 - a;
    }

    // Clear the interrupt Flag
    IFS0bits.IC1IF = 0;
  }
}

void __ISR(_INPUT_CAPTURE_2_VECTOR, ipl3) IC2Handler(void) {
  // Is this an IC2 interrupt?
  if (IFS0bits.IC2IF) {
    int a, b;
    // read data 2 times to clear interrupt
    a = IC2BUF; // oldest value
    b = IC2BUF; // newest value

    // check for timer overflow
    if (b > a) {
      pulse2 = b - a;
    }
    else {
      pulse2 = b + 65535 - a;
    }

    // Clear the interrupt Flag
    IFS0bits.IC2IF = 0;
  }
}

void __ISR(_INPUT_CAPTURE_3_VECTOR, ipl3) IC3Handler(void) {
  // Is this an IC1 interrupt?
  if (IFS0bits.IC3IF) {
    int a, b, c, d;
    // read data four times to clear interrupt
    a = IC3BUF; // oldest value
    b = IC3BUF; // newest value

    // check for timer overflow
    if (b > a) {
      pulse3 = b - a;
    }
    else {
      pulse3 = b + 65535 - a;
    }

    // Clear the interrupt Flag
    IFS0bits.IC3IF = 0;
  }
}

```