[Index](Index.md)

# Introduction #

UART with MATLAB

In this example, the PIC reads the USER button and analog voltage on pins B0 and B1 (analog A0 and A1) and streams the data to the app at 20Hz in the form "#user #A0 #A1\r\n".

100 times, or for 5 seconds, MATLAB reads the data, and draws a star at the (#A0/2,#A1/2) location on the plot, in blue if USER is not pushed, and red if USER is pushed. Every 20 data points (1 second) send a T to the PIC to toggle the LED.

## MATLAB Code ##
```
% open the serial port, read 100 data points (5 seconds at 20Hz)
% toggle the LED every second
% plot the data and close the port

% check to see if any ports are open, close them if they are
if ~isempty(instrfind)
    fclose(instrfind);
    delete(instrfind);
end

% open the port, change COM19 to match your port name
mySerial = serial('COM19', 'BaudRate', 230400, 'FlowControl', 'hardware');
fopen(mySerial);
disp('Reading...');

dataPoints = 0;
figure;
% read 100 messages
while (dataPoints < 100)
    % make sure there are bytes to read
    if(mySerial.BytesAvailable > 10)
        data = fscanf(mySerial, '%d %d %d');
        
        if (data(1) == 1) 
            plot(data(2),data(3),'b*');axis([0 1023 0 1023]);
        else
            plot(data(2),data(3),'r*');axis([0 1023 0 1023]);
        end
        % give some pause to see the figure
        pause(0.01);
        
        dataPoints = dataPoints + 1;
        if(mod(dataPoints,20) == 0)
            fwrite(mySerial,'T')
        end
    end
end

% close the port
fclose(mySerial);

disp('Compete');

% close the figure
close all;
```

## PIC Code ##
```
#include <plib.h>
#include "NU32.h"

void init_analog(void);

int main(void) {
  NU32_Startup();
  char message[50];
  int user = 0;
  int a0 = 0;
  int a1 = 0;

  // setup the ADC
  init_analog();

  // at 20Hz, read USER, a0 and a1 and send to the computer, and toggle LED2
  // if you get a 'T', toggle LED1
  while (1) {
    WriteCoreTimer(0);

    // read USER, A0 and A1
    a0 = ADC1BUF0;
    a1 = ADC1BUF1;
    user = NU32USER;

    // send the values
    sprintf(message, "%d %d %d\r\n",user,a0,a1);
    NU32_WriteUART1(message);

    // wait 1/30 sec
    while (ReadCoreTimer() < 40000000/20) {
      // is there a character from the computer?
      if(U1STAbits.URXDA) {
        char rx = U1RXREG;
        // toggle LED1 if you get a 'T'
        if(rx == 'T') {
          NU32LED1 = !NU32LED1;
        }
      }
    }
    NU32LED2 = !NU32LED2;
  }
  return 0;
}

void init_analog(void) {
  // Set all A0 and A1 pins (B0-B1) for analog input
  TRISBSET = 0xFFFF;
  AD1PCFGCLR = 0x3;

  AD1CON1bits.FORM = 0b000; // Select 16-bit integer output
  AD1CON1bits.ASAM = 1; // Auto Sample
  AD1CON1bits.SSRC = 0b111; // Auto convert sampled data to digital
  AD1CON2bits.VCFG = 0b000; // Select the vrefs to be Vdd and Vss

  AD1CON2bits.CSCNA = 1; // Enable ADC Scanning
  AD1CSSL = 0x3; // Select pins a0 and a1 for scanning
  AD1CON2bits.SMPI = 2-1; // Select 2 samples per interupt
  /* In scanning mode, this number must be at least as big as
   * the number of channels you are scanning/
   */

  AD1CON3bits.ADRC = 1; // Use internal RC clock
  AD1CON3bits.SAMC = 2; // Set sampling time 2*Tad
  AD1CON1bits.ADON = 1; // Turn the ADC on
}

```