[Index](Index.md)

# Introduction #

UART Loopback

**not loopback mode, as described in 21.11, which internally connects TX and RX. we will actually connect them with wire**

In order to test our code without an external device or another PIC, we will have the PIC talk to itself with two UART peripherals.



The wiring has RX to TX, so U2RX on F4 goes to U5TX on F13, and U2TX on F5 goes to U5RX on F12.

In the code, we set up each peripheral with a baud of 115200, no hardware flow control, and no interrupts.

## Code 1 ##
```
#include <plib.h>
#include "NU32.h"

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  char message[50];

  // init UART2
  // U2RX on F4, U2TX on F5
  U2MODEbits.BRGH = 0; // set baudrate to 115200
  U2BRG = ((SYS_FREQ / 115200) / 16) - 1;
  // 8 bit, no parity bit, and 1 stop bit (8N1 setup)
  U2MODEbits.PDSEL = 0;
  U2MODEbits.STSEL = 0;
  // configure TX & RX pins as output & input pins
  U2STAbits.UTXEN = 1;
  U2STAbits.URXEN = 1;
  // turn on the UART
  U2MODEbits.ON = 1;

  // init UART5
  // U5RX on F12, U5TX on F13
  U5MODEbits.BRGH = 0; // set baudrate to 115200
  U5BRG = ((SYS_FREQ / 115200) / 16) - 1;
  // 8 bit, no parity bit, and 1 stop bit (8N1 setup)
  U5MODEbits.PDSEL = 0;
  U5MODEbits.STSEL = 0;
  // configure TX & RX pins as output & input pins
  U5STAbits.UTXEN = 1;
  U5STAbits.URXEN = 1;
  // turn on the UART
  U5MODEbits.ON = 1;

  char rx2 = 0, rx5 = 0;

  // U2 sends a 2, U5 sends a 5
  // if all goes well, the LEDs are off
  // the debug COM will report the character received
  while (1) {
    // use UART2 to send a '2'
    while (U2STAbits.UTXBF); // wait until tx buffer isn't full
    U2TXREG = '2';

    // see if UART5 got a '2'
    while(!U5STAbits.URXDA) {} // wait until there is something to read
    rx5 = U5RXREG;

    if (rx5 == '2') {
      NU32LED1 = 1; // off
    }
    else {
      NU32LED1 = 0; // on
    }
    sprintf(message,"U5 got a %c\n",rx5); // %c so we get the ascii value
    NU32_WriteUART1(message);

    // use UART5 to send a '5'
    while (U5STAbits.UTXBF); // wait until tx buffer isn't full
    U5TXREG = '5';

    // see if UART2 got a '5'
    while(!U2STAbits.URXDA) {} // wait until there is something to read
    rx2 = U2RXREG;

    if (rx2 == '5') {
      NU32LED2 = 1; // off
    }
    else {
      NU32LED2 = 0; // on
    }
    sprintf(message,"U2 got a %c\n",rx2); // %c so we get the ascii value
    NU32_WriteUART1(message);

    // wait 1 sec
    WriteCoreTimer(0);
    while(ReadCoreTimer()<40000000) {}

  }
  return 0;
}
```