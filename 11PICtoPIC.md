[Index](Index.md)

# Introduction #

UART PIC to PIC communication

U5 set up with RX interrupt.

Send a 'U' at 5kHz. When received 5000 'U's, send a '!' to the computer.

Same code on two NU32s.

**Annoying that both debugging ports need to be opened on the computer, maybe switch to toggle LEDs**

## Code ##

```
#include <plib.h>
#include "NU32.h"

int gotRx = 0;

int main(void) {
  NU32_Startup();

  // init UART5
  // U5RX on F12, U5TX on F13
  U5MODEbits.BRGH = 0; // set baudrate to 230400
  U5BRG = ((SYS_FREQ / 230400) / 16) - 1;
  // 8 bit, no parity bit, and 1 stop bit (8N1 setup)
  U5MODEbits.PDSEL = 0;
  U5MODEbits.STSEL = 0;
  // configure TX & RX pins as output & input pins
  U5STAbits.UTXEN = 1;
  U5STAbits.URXEN = 1;

  // configure RX to interrupt whenever a character arrives
  U5STAbits.URXISEL = 0;

  // set priority levels & enable rx interrupt
  IPC12bits.U5IP = 1;
  IPC12bits.U5IS = 0;
  IEC2bits.U5RXIE = 1;

  // turn on the UART
  U5MODEbits.ON = 1;

  // send a 'U' at 5kHz
  // on the 5000th rx, send a '!' to the computer
  while (1) {
    WriteCoreTimer(0);

    // use UART5 to send a 'U'
    while (U5STAbits.UTXBF); // wait until tx buffer isn't full
    U5TXREG = 'U';

    // wait 200 micro sec
    while(ReadCoreTimer()<8000) {} // 40000000 / 0.0002s = 8000
  }
  return 0;
}

void __ISR(_UART_5_VECTOR, ipl1) IntUart5Handler(void) {
    // Is this an RX interrupt?
    if (IFS2bits.U5RXIF) {
        char rx5 = U5RXREG;

        if (rx5 == 'U') {
          gotRx++;

          if (gotRx == 5000) {
            PutCharacter(UART1, '!');
            gotRx = 0;
          }
        }
        // Clear the RX interrupt Flag
        IFS2bits.U5RXIF = 0;
    }
}
```