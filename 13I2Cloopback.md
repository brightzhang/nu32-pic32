[Index](Index.md)

# Introduction #

I2C loopback

## Library ##

```
#include <plib.h>
#include "NU32.h"
#include "NU32_I2C.h"

//Turn on I2C Module and calibrate for standard operation.
void I2Cinitialize(int baudrate){
//Configures I2CxADD register as a 7-bit slave address - not a 10 bit address
  I2C2CONbits.A10M = 1;
//sets Freq of SCLK to 171kHz
  I2C2BRG = baudrate;
//Enables the I2C module and configures SDA2 (A3) and SCL2 (A2) as serial pins
  I2C2CONbits.ON = 1;
}


//Turn on the ADC and calibrate for simple operation
void ADCinitialize(void){
    char readvalue = 0;
    //Reset all registers
    I2Cwrite(ADC_DEVICE_ADDRESS, ADC_CTRL_REGISTER, ADC_RESET);
    //Power up digital circuit
    I2Cwrite(ADC_DEVICE_ADDRESS, ADC_CTRL_REGISTER, ADC_START);
    //Read control register and alert if bit 3 is not high indicating power up
    //not ready
    I2Cread(ADC_DEVICE_ADDRESS, ADC_CTRL_REGISTER, &readvalue);
    if (!(readvalue & 0x08)){
      sprintf(NU32_RS232OutBuffer,"Device Not Powered Up\r\n");
      NU32_WriteUART1(NU32_RS232OutBuffer);
    }
    //Initialize for standard configuration
    I2Cwrite(ADC_DEVICE_ADDRESS, ADC_CTRL_REGISTER, ADC_INITIALIZE);
    sprintf(NU32_RS232OutBuffer,"ADC initialized\r\n");
    NU32_WriteUART1(NU32_RS232OutBuffer);
}

//This function sends one byte via I2C. It should only be called after a
//start event.
void I2Csendonebyte(char value){
    //check to see that the transmitter buffer is empty
    if(!I2C2STATbits.TBF){
       //Load the transmit register with the value to transmit
       I2C2TRN = value;
       //Check to ensure there was not a bus collision or a write collision
       if(I2C2STATbits.BCL | I2C2STATbits.IWCOL){
           sprintf(NU32_RS232OutBuffer,"Transmit Bus or Write Collision\r\n");
       NU32_WriteUART1(NU32_RS232OutBuffer);
       }
       //wait until transmission status is cleared at the end of transmisssion
       while(I2C2STATbits.TRSTAT){
       }
    //report error if transmit buffer was already full
    } else{
       sprintf(NU32_RS232OutBuffer,"Transmit Buffer Full\r\n");
       NU32_WriteUART1(NU32_RS232OutBuffer);
    }
    //Report if byte not acknowledged. This could indicate that the wrong byte
    //was sent or the SCLK and SDA lines were not hooked up correctly.
    if (I2C2STATbits.ACKSTAT){
       sprintf(NU32_RS232OutBuffer,"Byte not Acknowledged\r\n");
       NU32_WriteUART1(NU32_RS232OutBuffer);
    }
}

//This function reads one byte via I2C. It should only be used after a start
//event and the slave device have already been addressed as per I2C protocol
void I2Creadonebyte(char * variable){
    //Enable recieve
    I2C2CONbits.RCEN = 1;
    //Report if recieve events overlap.
    if(I2C2STATbits.I2COV){
       sprintf(NU32_RS232OutBuffer,"Data Overflow\r\n");
       NU32_WriteUART1(NU32_RS232OutBuffer);
    }
    //Wait until recieve register is full.
    while(!I2C2STATbits.RBF){
    }
    //Save recieved data to variable. Automatically clears RCEN and RBF
       *variable = I2C2RCV;
}

//Initiates a start event, master pulls SDA low hwile SCLK is high
void I2Cstartevent(void){
    //Check that the last event was a stop event which means the bus is idle
    if(!I2C2STATbits.S){
       //initiate start event
       I2C2CONbits.SEN = 1;
       //check for bus collisions
       if (I2C2STATbits.BCL){
          //if there was a collision report the error
          sprintf(NU32_RS232OutBuffer,"Start Event Collision\r\n");
          NU32_WriteUART1(NU32_RS232OutBuffer);
      }
      //wait until the end of the start event clears the start enable bit
      while(I2C2CONbits.SEN){
      }
    }
    else{
          //if the bus was not idle report the error
          sprintf(NU32_RS232OutBuffer,"Bus is not idle\r\n");
          NU32_WriteUART1(NU32_RS232OutBuffer);
    }
}

//Initiates a repeat start event
void I2Crepeatstartevent(void){
 //Initiates a repeat start event, master pulls SDA low
    I2C2CONbits.RSEN = 1;
    if (I2C2STATbits.BCL){
          sprintf(NU32_RS232OutBuffer,"Repeat Start Event Collision\r\n");
          NU32_WriteUART1(NU32_RS232OutBuffer);
    }
    //wait until the end of the repeatstart event clears the repeat
    //start enable bit
    while(I2C2CONbits.RSEN){
    }
}

//Initiates a stop event to end transmission and leave bus idle
void I2Cstopevent(void){
    //Initiates a stop event, master sets SDA high on a high SCLK
    I2C2CONbits.PEN = 1;
    //wait for stop event enable to be cleared by completion of stop event
    while(I2C2CONbits.PEN){
    }
}

//This function initiates a full transmission to write a single byte to
//a register of a slave device via I2C. The device address is the 7 bit
//address of the slave device with a 0 as the least significant bit.
void I2Cwrite(char deviceaddress, char registeraddress, char value){

    I2Cstartevent();

    I2Csendonebyte(deviceaddress);
    I2Csendonebyte(registeraddress);
    I2Csendonebyte(value);

    I2Cstopevent();
}

//This function initiates a full transmission to read a single byte from
//a register of a slave device via I2C.The device address is the 7 bit
//address of the slave device with a 0 as the least significant bit.
void I2Cread(char writeaddress, char registeraddress, char * value){

    char readaddress = writeaddress + 0x01;
    I2Cstartevent();
    I2Csendonebyte(writeaddress);
    I2Csendonebyte(registeraddress);

    I2Crepeatstartevent();
    I2Csendonebyte(readaddress);
    I2Creadonebyte(value);
    I2Cstopevent();
}

//This function initiates a full transmission to read three consecutive bytes of
//a slave device via I2C. The device address is the 7 bit address of the slave
//device with a 0 as the least significant bit.
void I2Cburstread(char writeaddress, char firstregisteraddress, int * value){
    char value1 = 0;
    char value2 = 0;
    char value3 = 0;
    char readaddress = writeaddress + 0x01;

    I2Cstartevent();
    I2Csendonebyte(writeaddress);
    I2Csendonebyte(firstregisteraddress);

    I2Crepeatstartevent();
    I2Csendonebyte(readaddress);
    I2Creadonebyte(&value1);
    I2Creadonebyte(&value2);
    I2Creadonebyte(&value3);
    I2Cstopevent();
    *value = (value1<<16) + (value2<<8) + (value3);
}

void I2Cslaveinitialize(char slaveaddress){
  I2C2ADD = slaveaddress;
//Configures I2CxADD register as a 7-bit slave address - not a 10 bit address
  I2C1CONbits.A10M = 1;
//Enables the I2C module and configures SDA2 (A3) and SCL2 (A2) as serial pins
  I2C1CONbits.ON = 1;
}
```

```
#ifndef __NU32I2C_H
#define __NU32I2C_H

//Defines
//Register addresses and for MAX518
#define DAC_DEVICE_ADDRESS 0x5E
#define DAC_D0_ADDRESS 0x00

//Register addresses and configuration bits for NAU7802
#define ADC_DEVICE_ADDRESS  0b01010100
#define ADC_CTRL_REGISTER  0x00
#define ADC_RESET  0x01
#define ADC_START  0x02
#define ADC_INITIALIZE  0x3E
#define ADC_RESULT_ADDRESS  0x12

#define SRAM_DEVICE_ADDRESS 0b10101110
//create a clock freq of about 95 kHz
#define SLOW_BAUD_RATE 240

//prototypes for functions
void I2Cinitialize(int baudrate);
void I2Cwrite(char deviceaddress, char registeraddress, char value);
void I2Creadonebyte(char * variable);
void I2Csendonebyte(char value);
void I2Cstartevent(void);
void I2Crepeatstartevent(void);
void I2Cstopevent(void);
void I2Cread(char writeaddress, char registeraddress, char * value);
void I2Cburstread(char writeaddress, char firstregisteraddress, int * value);
void ADCinitialize();
void I2Csecondread(char writeaddress, char * value);

#endif // __NU32_H
```


## Code ##

```
Code here
```