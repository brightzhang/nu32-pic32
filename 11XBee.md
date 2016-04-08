[Index](Index.md)

# Introduction #

UART with XBee

Out of the box, an XBee uses a baud of 9600.

On the XBee, pin 1 is 3.3V, pin 10 is gnd. Pin 2 is Tx and pin 3 is Rx.

On the NU32, UART2 is used. U2RX is F4 and U2TX is F5.

Put this code on two identical NU32 circuits.

Each NU32 will send the character '2', and then wait 1 second, checking to see if a '2' is received. If it is, LED1 is toggled. If a character is received, but it is not a '2', LED2 is toggled.

XBee properties can be changed using a USB to UART cable with the X-CTU software from Digi (http://www.digi.com/support/productdetail?pid=3352&osvid=57&type=utilities)

or

using commands sent via UART from chapter 3 of the XBee datasheet (https://www.sparkfun.com/datasheets/Wireless/Zigbee/XBee-Datasheet.pdf)


## Code ##

```
#include <plib.h>
#include "NU32.h"

int main(void) {
  NU32_Startup();
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  // init UART2
  // U2RX on F4 to XBee TX pin 2
  // U2TX on F5 to Xbee RX pin 3
  U2MODEbits.BRGH = 0; // set baudrate to 115200
  U2BRG = ((SYS_FREQ / 9600) / 16) - 1;
  // 8 bit, no parity bit, and 1 stop bit (8N1 setup)
  U2MODEbits.PDSEL = 0;
  U2MODEbits.STSEL = 0;
  // configure TX & RX pins as output & input pins
  U2STAbits.UTXEN = 1;
  U2STAbits.URXEN = 1;
  // configure RX to interrupt whenever a character arrives
  U2STAbits.URXISEL = 0; // if you turn on the interrupt
  // finally, turn on the UART
  U2MODEbits.ON = 1;

  while (1) {
    while (U2STAbits.UTXBF); // wait until tx buffer isn't full
    U2TXREG = '2';

    // wait 1 sec, see if you get anything while waiting
    WriteCoreTimer(0);
    while(ReadCoreTimer()<40000000) {
      if(U2STAbits.URXDA) {
        char rx = U2RXREG;
        if(rx == '2') {
          NU32LED1 = !NU32LED1;
        }
        else {
          NU32LED2 = !NU32LED2;
        }
      }
    }
  }
  return 0;
}
```