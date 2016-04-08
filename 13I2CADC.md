[Index](Index.md)

# Introduction #

I2C to NAU7802 ADC


## Code ##

```
#include <plib.h>
#include "NU32.h"
#include "NU32_I2C.h"

/* This program is intended to run with the NAU7802 24 bit dual channel
* analog to digital converter. It is intended as sample code to demonstrate
* read and write operations of the I2C serial protocol. In order to
* obtain reliable data from the ADC it will be necessary to calibrate the
* NAU7802 chip more than this code.
* To use this program include NU32.c , NU23.h , NU32_I2C.c , NU32_I2C.h in your
* project
* This program uses I2C module 2 on the pic, which uses SCLK: A3, SDA: A4.
*
* The setup for  the NAU7802 is shown below:
*
*             	___________
*       	3.3V | 1   U  16 | 3.3V
*        	GND | 2  	15 | 3.3V
* 	Voltage IN | 3  	14 | SDA (A3)
*            	| 4  	13 | SCLK (A2)
*            	| 5  	12 |
*            	| 6  	11 |
*        	GND | 7  	10 |
*        	GND | 8   	9 | GND
*             	-----------
*
*
*////////////////////////////////////////////////////////////////////////
int main() {
// NU32 initialization:
NU32_Startup();
NU32_Initialize();
//Turn on and initialize I2C module 2
I2Cinitialize(SLOW_BAUD_RATE);
//Send bytes to ADC to turn on and initialize
ADCinitialize();

int i = 0;
int j = 0;
int intvalue = 0;
while (1) {

	if(!NU32USER){
    	//Turn off LED1 until transfer it complete
    	NU32LED1 = 1;
    	I2Cburstread(ADC_DEVICE_ADDRESS, ADC_RESULT_ADDRESS, &intvalue);
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