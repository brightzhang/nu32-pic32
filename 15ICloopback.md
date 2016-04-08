[Index](Index.md)

# Introduction #

Input capture loopback

To test the IC peripheral, create a PWM with a known duty cycle. Feed the PWM back into the IC pin, and read the duty cycle to see if it matches what was output.

Connect OC1 on D0 to IC1 on D8. Use IC1 to read the duty cycle of D0. Every 100th read, send the output duty cycle and the read duty cycle to the computer to compare, and change the output duty cycle.

## Code ##
```
#include <plib.h>
#include "NU32.h"
#include <math.h>

int pulse = 0, period = 0, count = 0, flag = 0, i = 0;

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  char message[100];

  // set up OC1 on D0
  T2CON = 0; // turn off T2
  OC1CON = 0; // turn off OC1
  T2CONbits.TCKPS = 7; // set prescaler 1:256
  TMR2 = 0; // start T2 at 0
  PR2 = 1000; // period of 3.2mS = PR2 + 1 / PBCK * PS
  OC1CONbits.OCM = 0b110; // PWM mode with no fault protection
  OC1RS = 10; // set buffered PWM duty cycle in counts,
  OC1R = 10; // set initial PWM duty cycle in counts
  T2CONbits.ON = 1; // turn on TMR2
  OC1CONbits.ON = 1; // turn on OC1

  // set up Input Capture 1 on D8 with T3
  T3CONbits.TCKPS = 5; // set prescaler 1:32
  TMR3 = 0; // start T3 at 0
  PR3 = 65535; // max period match = 26.2mS
  IC1CONbits.C32 = 0; // use 16 bit
  IC1CONbits.ICTMR = 0; // use T3
  IC1CONbits.FEDGE = 1; // capture rising edge first
  IC1CONbits.ICM = 6; // capture every edge starting with the above
  IC1CONbits.ICI = 3; // interrupt on every 4th capture event
  IPC1bits.IC1IP = 3; // interrupt priority level 3
  IPC1bits.IC1IS = 0; // subpriority 0
  IEC0bits.IC1IE = 1; // enable IC1 interrupt
  IC1CONbits.ON = 1; // turn on IC1
  T3CONbits.ON = 1; // turn on T3

  // after 100 interrupts, send the recorded duty cycle and output duty cycle
  // to the computer and increment the output duty cycle
  while (1) {
    if (flag) {
      sprintf(message,"pulse = %d, period = %d, duty = %d, OC1RS = %d\r\n", pulse, period, pulse*1000/period, i);
      NU32_WriteUART1(message);
      
      flag = 0; // reset the 100 interrupt flag

      // increment the output duty cycle
      i = i + 25;
      // 1000 is the max value of PR2
      if (i >= 1000) {
        i = 10;
      }
      OC1RS = i;
    }
  }

  return 0;
}

// on every fourth capture, calculate the pulse length and period
void __ISR(_INPUT_CAPTURE_1_VECTOR, ipl3) IC1Handler(void) {
  // Is this an IC1 interrupt?
  if (IFS0bits.IC1IF) {
    int a, b, c, d;
    // read data four times to clear the buffer
    a = IC1BUF; // first value
    b = IC1BUF; // second value
    c = IC1BUF; // third value
    d = IC1BUF; // fourth value

    // the difference from a to b is the pulse length
    // the difference from a to c is the period
    pulse = b - a;
    period = c - a;

    // reset T3 to avoid overflow issues
    TMR3 = 0;

    // note how many interrupts have happened
    // if 100 have occurred, set the flag to output the
    // pulse and period to the computer
    count++;
    if(count >= 100) {
      flag = 1;
      count = 0;
    }

    // Clear the interrupt Flag
    IFS0bits.IC1IF = 0;
  }
}

```