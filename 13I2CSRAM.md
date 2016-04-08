[Index](Index.md)

# Introduction #

I2C to SRAM


## Code ##

```
#include <plib.h>
#include "NU32.h"
#include "NU32_I2C.h"

/* This program is intended to run with the PCF8570 256 x 8 bit RAM.
* It is intended as sample code to demonstrate read and write operations of
* the I2C serial protocol.
* To use this program include NU32.c , NU23.h , NU32_I2C.c , NU32_I2C.h in your
* project
* This program uses I2C module 2 on the pic, which uses SCLK: A3, SDA: A4.
*
* The setup for  the PCF8570 is shown below:
*
*             	___________
*       	3.3V | 1   U  8 | 3.3V
*       	3.3V | 2  	7 |
*       	3.3V | 3  	6 | SCLK (A2)
*        	GND | 4  	5 | SDA (A3)
*             	-----------
*
*////////////////////////////////////////////////////////////////////////
int main() {
// NU32 initialization:
NU32_Startup();
NU32_Initialize();
//Turn on and initialize I2C module 2
I2Cinitialize(SLOW_BAUD_RATE);

int i = 0;
int j = 0;
char value = 0;
int intvalue = 0;
while (1) {

	if(!NU32USER){
    	//Turn off LED1 until transfer it complete
    	NU32LED1 = 1;
    	//send the data byte 0x1F to the register 0x01
    	I2Cwrite(SRAM_DEVICE_ADDRESS, 0x01, 0x1F);
    	//read back register 0x01
    	I2Cread(SRAM_DEVICE_ADDRESS, 0x01, &value);
    	//Convert char to int and get rid of the sign bit
    	intvalue = (int)value & 0xFF;
    	sprintf(NU32_RS232OutBuffer,"Data: %d \r\n", intvalue);
    	NU32_WriteUART1(NU32_RS232OutBuffer);
    	//Wait to prevent switch bounce from retriggering
    	while(i<5000000){
        	i++;
    	}
    	i =0;
    	NU32LED1 = 0;
	}
	//Flash LED2 to show program is still running
	if (j >= 5000000){
    	NU32LED2 = !NU32LED2;
    	j = 0;
	}
	j++;

}
return;
}
```