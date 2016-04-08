[Index](Index.md)

# Introduction #

UART with hardware flow control

UART2 is connected to an FTDI cable with the hardware flow control pins.

Use PUTTY or other serial terminal program to open the FTDI port at 115200, using RTS/CTS option.

Type an s. The s will be returned, along with the numbers 1 to 1000.

## Code ##

```
#include <plib.h>
#include "NU32.h"

int rxFlag = 0;

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  char message[50];
  int i = 0;

  // init UART2
  // U2RX on F4, U2TX on F5, U2RTS on F13, U2CTS on F12
  // FTDI cable Black to GND, Yellow is RX, Orange is TX, Brown is CTS, Green is RTS
  // remember RX goes to TX, RTS goes to CTS
  U2MODEbits.BRGH = 0; // set baudrate to 115200
  U2BRG = ((SYS_FREQ / 115200) / 16) - 1;
  // 8 bit, no parity bit, and 1 stop bit (8N1 setup)
  U2MODEbits.PDSEL = 0;
  U2MODEbits.STSEL = 0;
  // configure TX & RX pins as output & input pins
  U2STAbits.UTXEN = 1;
  U2STAbits.URXEN = 1;
  // configure using RTS and CTS
  U2MODEbits.UEN = 2;

  // configure RX to interrupt whenever a character arrives
  U2STAbits.URXISEL = 0;

  // set priority levels & enable rx interrupt
  IPC8bits.U2IP = 1;
  IPC8bits.U2IS = 0;
  IEC1bits.U2RXIE = 1;

  // turn on the UART
  U2MODEbits.ON = 1;

  // if the rxFlag is set, send the numbers 1-1000 to the computer
  // as fast as you can
  while (1) {
    if (rxFlag) {
      for (i = 1; i < 1001; i++) {
        sprintf(message, "%d\r\n", i);
        WriteString(UART2, message);
      }

      // reset the flag
      rxFlag = 0;
    }
  }
  return 0;
}

void __ISR(_UART_2_VECTOR, ipl1) IntUart2Handler(void) {
  // Is this an RX interrupt?
  if (IFS1bits.U2RXIF) {
    char rx2 = U2RXREG;

    if (rx2 == 's') {
      // echo the 's' so the user has feedback
      PutCharacter(UART2, 's');
      PutCharacter(UART2, '\r');
      PutCharacter(UART2, '\n');

      // set the flag
      rxFlag = 1;
    }
    // Clear the RX interrupt Flag
    IFS1bits.U2RXIF = 0;
  }
}
```