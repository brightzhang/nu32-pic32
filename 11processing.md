[Index](Index.md)

# Introduction #

UART with Processing

[Processing](http://processing.org/) is a free IDE for creating visual programs in Java.

In this example, the PIC reads the USER button and analog voltage on pins B0 and B1 (analog A0 and A1) and streams the data to the app at 20Hz in the form "#user #A0 #A1\r\n". The app draws a circle at the (#A0/2,#A1/2) location on screen, in blue if USER is not pushed, and red if USER is pushed.

Processing does not have a RTS/CTS option for turning on hardware flow control, so the PIC code has to be altered to not use flow control. In NU32.c UART1, change UART\_ENABLE\_PINS\_CTS\_RTS to UART\_ENABLE\_PINS\_TX\_RX\_ONLY

## Processing Code ##
```
import processing.serial.*;
Serial myPort; // The serial port
 
// Incoming serial data
int x = 0;
int y = 0;
int user = 0;

void setup() {
  size(512, 512);
  // List all the available serial ports:
  println(Serial.list());
  // Open the port you're using
  String portName = Serial.list()[1]; // in my case, the second in the list
  myPort = new Serial(this, portName, 230400); // no option for RTS/CTS, so change NU32.c
  myPort.bufferUntil('\n'); // buffer all data, call serial event when you get a newline
}

void draw() {
  background(0); // black background
  if (user == 1) {
    fill(0,0,255); // blue
  }
  else {
    fill(255,0,0); // red
  }
  ellipse(x, y, 10, 10); // dot
}

// when the port gets a '\n'
void serialEvent(Serial myPort) {
  String inString = (myPort.readString()); // get the buffer
  inString = inString.trim(); // remove excess spaces
  int[] values = int(split(inString, ' ')); // turn into an array of integers
  // error check, make sure there are 3 ints
  if (values.length == 3) {
    user = values[0];
    x = values[1]/2;
    y = values[2]/2;
  }
}

// if the mouse is pressed, send a T to toggle the LED
void mousePressed() {
  myPort.write('T');
}

```

## PIC Code ##
```
#include <plib.h>
#include "NU32.h"

void init_analog(void);

int main(void) {
  // turn off the flow control to work with Processing
  NU32_Startup(); // In NU32.c UART1, change UART_ENABLE_PINS_CTS_RTS to UART_ENABLE_PINS_TX_RX_ONLY
  char message[50];
  int user = 0;
  int a0 = 0;
  int a1 = 0;

  // setup the ADC
  init_analog();

  // at 20Hz, read USER, a0 and a1 and send to the computer, and toggle LED2
  // if you get a 'T', toggle LED1
  while (1) {
    WriteCoreTimer(0);

    // read USER, A0 and A1
    a0 = ADC1BUF0;
    a1 = ADC1BUF1;
    user = NU32USER;

    // send the values
    sprintf(message, "%d %d %d\r\n",user,a0,a1);
    NU32_WriteUART1(message);

    // wait 1/30 sec
    while (ReadCoreTimer() < 40000000/20) {
      // is there a character from the computer?
      if(U1STAbits.URXDA) {
        char rx = U1RXREG;
        // toggle LED1 if you get a 'T'
        if(rx == 'T') {
          NU32LED1 = !NU32LED1;
        }
      }
    }
    NU32LED2 = !NU32LED2;
  }
  return 0;
}

void init_analog(void) {
  // Set all A0 and A1 pins (B0-B1) for analog input
  TRISBSET = 0xFFFF;
  AD1PCFGCLR = 0x3;

  AD1CON1bits.FORM = 0b000; // Select 16-bit integer output
  AD1CON1bits.ASAM = 1; // Auto Sample
  AD1CON1bits.SSRC = 0b111; // Auto convert sampled data to digital
  AD1CON2bits.VCFG = 0b000; // Select the vrefs to be Vdd and Vss

  AD1CON2bits.CSCNA = 1; // Enable ADC Scanning
  AD1CSSL = 0x3; // Select pins a0 and a1 for scanning
  AD1CON2bits.SMPI = 2-1; // Select 2 samples per interupt
  /* In scanning mode, this number must be at least as big as
   * the number of channels you are scanning/
   */

  AD1CON3bits.ADRC = 1; // Use internal RC clock
  AD1CON3bits.SAMC = 2; // Set sampling time 2*Tad
  AD1CON1bits.ADON = 1; // Turn the ADC on
}

```