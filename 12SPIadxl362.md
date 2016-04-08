[Index](Index.md)

# Introduction #

Using SPI to talk to the ADXL362 accelerometer

Spec sheet for the [ADXL362](http://www.analog.com/static/imported-files/data_sheets/ADXL362.pdf)

A [breakout board from Sparkfun](https://www.sparkfun.com/products/11446) $14.95

The ADXL362 is a 3 axis, 12 bit accelerometer and temperature sensor, that communicates using SPI.

The ADXL362 has many cool features, like "wake-on-shake", and is super low power.

This code will initialize the ADXL362 for plus/minus 2g sensitivity. Read the spec sheet to learn how to set up the more advanced features.

## Details ##

Using SPI1, connect SCK to SCK1 on D10, MOSI to SDO1 on D0, MISO to SDI1 on C4, and CS to D9 (this is SS1, but it will be controlled manually, not by the peripheral).

## Code ##

```
//#define NU32_STANDALONE  // uncomment if program is standalone, not bootloaded
#include "NU32.h"          // plib.h, config bits, constants, funcs for startup and UART
#define MAX_MESSAGE_LENGTH 200

// Read the acceleration and temperature from the ADXL362 accelerometer
// http://www.analog.com/static/imported-files/data_sheets/ADXL362.pdf

// Using SPI1:
//  - SCK on D10
//  - SDI (MISO) on C4
//  - SDO (MOSI) on D0
// SS1 on D9 is used for CS but is controlled as an IO pin, not by the peripheral

// manually control the slave select pin
#define CS LATDbits.LATD9

// functions
void initializeADXL362SPI();
void initializeADXL362();
void readAllRegisters();
void readXYZT(int * x, int * y, int * z, int * t);

int main(void) {
  char message[MAX_MESSAGE_LENGTH];

  NU32_Startup(); // cache on, min flash wait, interrupts on, LED/button init, UART init
  NU32LED1 = 1; // off
  NU32LED2 = 1; // off

  initializeADXL362SPI();
  initializeADXL362();

  // double check that the registers were updated, they should be:
  /*
   0x20 = 0
   0x21 = 0
   0x22 = 0
   0x23 = 0
   0x24 = 0
   0x25 = 0
   0x26 = 0
   0x27 = 0
   0x28 = 0
   0x29 = 0x80
   0x2A = 0
   0x2B = 0
   0x2C = 0x10
   0x2D = 0x22
   0x2E = 0
   */
  readAllRegisters();

  int x, y, z, t;
  double fx, fy, fz, ft;

  // read the accelerometer
  // and output to the computer 10Hz
  while (1) {
    WriteCoreTimer(0);

    readXYZT(&x, &y, &z, &t);

    // convert data to g using the sensitivity (plus/minus 2g over 12bit)
    fx = x / 2048.0 * 2.0;
    fy = y / 2048.0 * 2.0;
    fz = z / 2048.0 * 2.0;
    // convert temperature to degree F
    ft = (t * .065) * 9.0 / 5.0 + 32.0;

    sprintf(message, "%6.3f\t %6.3f\t %6.3f\t %6.3f\r\n", fx, fy, fz, ft);
    NU32_WriteUART1(message);

    while (ReadCoreTimer() < 4000000) {}
  }
  return 0;
}

void initializeADXL362SPI() {
  // setup the slave select pin on D9
  CS = 1;
  TRISDbits.TRISD9 = 0;

  // init SPI1
  SPI1CON = 0; // Disable and reset SPI module
  SPI1BRG = 9; // Set the baudrate to (80MHz)/2/(9+1) = 4Mhz
  SPI1STATbits.SPIROV = 0; // Clear overflow flag
  SPI1CONbits.MSTEN = 1; // Enable Master Mode
  SPI1CONbits.MODE32 = 0;
  SPI1CONbits.MODE16 = 0; // Data is 8 bits wide
  // for SPI type CPHA = CPOL = 0, also called MODE0
  SPI1CONbits.CKP = 0; // Clock is active high, idle low
  SPI1CONbits.CKE = 0; // Serial Data changes on idle to active
  SPI1CONbits.SSEN = 0; // don't use the SS pin automatically, control it in code
  SPI1CONbits.MSSEN = 0; // don't use the SS pin functionality
  SPI1CONbits.ON = 1; // Turn the SPI module on
}

void initializeADXL362() {
  int tempRead;

  // initialize the ADXL362 by doing a soft reset
  // write 0x1F 0x52
  CS = 0;
  SPI1BUF = 0x0A;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  SPI1BUF = 0x1F;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  SPI1BUF = 0x52;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  CS = 1;

  // wait for reset to complete
  WriteCoreTimer(0);
  while (ReadCoreTimer() < 400000) {} // 1ms

  // set the sensitivity to p/m 2g, 12.5Hz sampling, 1/4 sampling freq filter
  // write 0x2C 0x10
  CS = 0;
  SPI1BUF = 0x0A;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  SPI1BUF = 0x2C;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  SPI1BUF = 0x10;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  CS = 1;

  // write to the power control register, tell the chip to:
  // use ultralow noise, internal clock, normal operation, not wakeup mode
  // no autosleep, measurement mode
  // write 0x2D 0x22
  CS = 0;
  SPI1BUF = 0x0A;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  SPI1BUF = 0x2D;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  SPI1BUF = 0x22;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it
  CS = 1;
}

void readAllRegisters() {
  char message[50];
  int tempRead;
  int r20, r21, r22, r23, r24, r25, r26, r27, r28, r29, r2a, r2b, r2c, r2d, r2e;

  // read from register 0x20 to 0x2E
  CS = 0;
  SPI1BUF = 0x0B;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x20;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r20 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r21 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r22 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r23 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r24 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r25 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r26 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r27 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r28 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r29 = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r2a = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r2b = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r2c = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r2d = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  r2e = SPI1BUF; // Read the buffer to clear it
  CS = 1;

  // send the register values to the computer

  sprintf(message, "Registers:\r\n");
  NU32_WriteUART1(message);

  sprintf(message, "0x20: %x\r\n", r20);
  NU32_WriteUART1(message);

  sprintf(message, "0x21: %x\r\n", r21);
  NU32_WriteUART1(message);

  sprintf(message, "0x22: %x\r\n", r22);
  NU32_WriteUART1(message);

  sprintf(message, "0x23: %x\r\n", r23);
  NU32_WriteUART1(message);

  sprintf(message, "0x24: %x\r\n", r24);
  NU32_WriteUART1(message);

  sprintf(message, "0x25: %x\r\n", r25);
  NU32_WriteUART1(message);

  sprintf(message, "0x26: %x\r\n", r26);
  NU32_WriteUART1(message);

  sprintf(message, "0x27: %x\r\n", r27);
  NU32_WriteUART1(message);

  sprintf(message, "0x28: %x\r\n", r28);
  NU32_WriteUART1(message);

  sprintf(message, "0x29: %x\r\n", r29);
  NU32_WriteUART1(message);

  sprintf(message, "0x2a: %x\r\n", r2a);
  NU32_WriteUART1(message);

  sprintf(message, "0x2b: %x\r\n", r2b);
  NU32_WriteUART1(message);

  sprintf(message, "0x2c: %x\r\n", r2c);
  NU32_WriteUART1(message);

  sprintf(message, "0x2d: %x\r\n", r2d);
  NU32_WriteUART1(message);

  sprintf(message, "0x2e: %x\r\n", r2e);
  NU32_WriteUART1(message);
}

void readXYZT(int * x, int * y, int * z, int * t) {
  int xHigh = 0, xLow = 0, yHigh = 0, yLow = 0, zHigh = 0, zLow = 0, tHigh = 0, tLow = 0;
  int tempRead, tempX, tempY, tempZ, tempT;

  // read from 0x0E to 0x15
  CS = 0;
  SPI1BUF = 0x0B;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x0E;
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tempRead = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  xLow = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  xHigh = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  yLow = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  yHigh = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  zLow = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  zHigh = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tLow = SPI1BUF; // Read the buffer to clear it

  SPI1BUF = 0x00; // write garbage
  while (!SPI1STATbits.SPIRBF); // Wait for transaction to finish
  tHigh = SPI1BUF; // Read the buffer to clear it
  CS = 1;

  // the 4 LSB bits in High have the 4 MSB bits in the 12 bit value
  // the 4 MSB bits in High have the sign of the value

  tempX = ((xHigh & 0b00001111) << 8) | xLow;
  if ((xHigh & 0b00001000) == 0b0001000) {
    tempX = tempX - 4096;
  }

  tempY = ((yHigh & 0b00001111) << 8) | yLow;
  if ((yHigh & 0b00001000) == 0b0001000) {
    tempY = tempY - 4096;
  }

  tempZ = ((zHigh & 0b00001111) << 8) | zLow;
  if ((zHigh & 0b00001000) == 0b0001000) {
    tempZ = tempZ - 4096;
  }

  tempT = ((tHigh & 0b00001111) << 8) | tLow;

  *x = tempX;
  *y = tempY;
  *z = tempZ;
  *t = tempT;
}
```