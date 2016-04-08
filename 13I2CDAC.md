[Index](Index.md)

# Introduction #

I2C to MAX518 DAC

## Code ##

```
#include <plib.h>
#include "NU32.h"
#include "NU32_I2C.h"

/*
  * This function sends a single byte of data to the MAX518
  * digital to analog converter setting the analog voltage output from
  * D0 (pin 1, closest to divot) to the associated voltage:
  *	0xFF = Vin
  *	0x00 = GND
  * This code assumes the following pin connections where Vin<=6V :
  *	Pin 1: Voltage Out
  *	Pin 2: GND
  *	Pin 3: SCL2 (A2) with 2.2k resistor to Vin
  *	Pin 4: SDA2 (A3) with 2.2k resistor to Vin
  *	Pin 5: Vin (3.3V)
  *	Pin 6: Vin (3.3V)
  *	Pin 7: Vin (3.3V)
  *	Pin 8: D1 No Connection
  *
 ****************************************************************/

int main() {
 // NU32 initialization:
 NU32_Startup();
 NU32_Initialize();

 // Initialize the I2C module2. SCL2: pin A2, SDA2 pin A3
 I2Cinitialize(SLOW_BAUD_RATE);

 // Create an incrementing variable to send to the DAC to increment
 // the output voltage
 char count = 0x0F;
 int i = 0;
 int j = 0;
 while (1) {

 	if(!NU32USER){
     	//Turn off LED1 until send it complete
     	NU32LED1 = 1;
     	I2Cwrite(DAC_DEVICE_ADDRESS, DAC_D0_ADDRESS, count);
     	if(count <= 0xFF){
       	count = count + 0x10;
     	} else {
         	count = 0x00;
     	}
     	while(i<5000000){
         	i++;
     	}
     	i =0;
     	NU32LED1 = 0;
 	}
 	//Flash LED2
 	if (j >= 5000000){
     	NU32LED2 = !NU32LED2;
     	j = 0;
 	}
 	j++;

 }
 return;
}
```