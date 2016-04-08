[Index](Index.md)

# Introduction #

SPI loopback test


## Code ##

```
/*
 * SPI Sample Code, Loopback
 * Lyndon Sapozhnik,  12/12/2012
 * Updated 2/7/2013 for the 2013 NU32 Board
 *
 * Configures SPI4 as a master, SPI1 as a slave on the same PIC32
 *
 * When the USER button is pressed, the master sends the character 'A' to the
 * slave and the slave sends the character 'B' to the master. What the master
 * received and what the slave received is then displayed for the user to verify
 *
 * Connect these pins: SDO4 -> SDI1 (pin F5 -> pin C4)
 *                 	SDO1 -> SDI4 (pin D0 -> pin F4)
 *                 	SCK4 -> SCK1 (pin F13 -> pin D10)
*/
 
#include <plib.h>
#include "NU32.h"
 
// Function declarations
void dataTransfer(void);
 
int main(void)
{
	// setup NU32 LED's and buttons
	NU32_Startup();
	
	// Master - SPI4, pins are: SDI4(F4), SDO4(F5), SCK4(F13)
	int data = 0;
	SPI4CON = 0; // stop and reset SPI4
	data = SPI4BUF; // clear the rx buffer
	SPI4BRG = 0x4; // bit rate to 8MHz [SPI4BRG = (80000000/(2*desired))-1]
	SPI4STATbits.SPIROV = 0; // clear overflow
 
	// initalize SPI4 register
	SPI4CONbits.ON = 1;	// SPI4 ON
	SPI4CONSET = 00 << 10; // 8 bit xfer [1x = 32bit, 01 = 16bit, 00 = 8bit]
	SPI4CONbits.SMP = 1;   // SMP=1 [input data sampled at end of data output time]
	SPI4CONbits.MSTEN = 1; // Master
 
	// Slave - SPI1, pins are: SDI1(C4), SDO1(D0), SCK1(D10), SS1(D9)
	SPI1CON = 0;	// stop and reset SPI1
	data = SPI1BUF; // clear the rx buffer
	SPI1BRG = 0x4;  // bit rate to 8MHz [SPI4BRG = (80000000/(2*desired))-1]
	SPI1STATbits.SPIROV = 0; // clear overflow
 
	// initalize SPI1 register
	SPI1CONbits.ON = 1;	// SPI ON
	SPI1CONSET = 00 << 10; // 8 bit xfer [1x = 32bit, 01 = 16bit, 00 = 8bit]
	SPI1CONbits.SMP = 1;   // SMP=1 [input data sampled at end of data output time]
	SPI1CONbits.MSTEN = 0; // Slave
	LATDbits.LATD9 = 1;	// tie D9 (Slave Select 1 pin) high keep slave deselected
 
	while(1){
 	if (!NU32USER)
     	dataTransfer();	// transfer data when USER is pressed down
	}
	return 0;
}
 
void dataTransfer() {
  char message[32];
  int dataMaster = 0, dataSlave = 0;
  LATDbits.LATD9 = 0;       	// tie slave select low to select it
  SPI4BUF = 'A';            	// master sends the character 'A'
  while (!SPI1STATbits.SPIRBF); // wait until receive buffer is full
  dataSlave = SPI1BUF;      	// clear the rx buffer
  SPI1BUF = 'B';            	// slave sends the character 'B'
  while (!SPI4STATbits.SPIRBF); // wait until receive buffer is full
  dataMaster = SPI4BUF;     	// clear the rx buffer
  LATDbits.LATD9 = 1;       	// tie slave select high again
 
  // print data received by master, data received by slave
  sprintf(message,"Master:%c	Slave:%c\r\n",dataMaster,dataSlave);
  NU32_WriteUART1(message);
}

```