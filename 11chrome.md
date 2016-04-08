[Index](Index.md)

# Introduction #

UART with Chrome app

You can create a stand-alone application launched from the Chrome browser that can communicate with a [serial port](http://developer.chrome.com/apps/serial.html).

In this example, the PIC reads the USER button and analog voltage on pins B0 and B1 (analog A0 and A1) and streams the data to the app at 20Hz in the form "#user #A0 #A1\r\n". The app draws a circle at the (#A0/2,#A1/2) location on the HTML5 canvas, in blue if USER is not pushed, and red if USER is pushed. This is the same PIC code as in [PIC to Processing](11processing.md) and [PIC to MATLAB](11matlab.md).

The chrome app cannot print all of the values without lagging when the data streams in at faster than 30Hz on my computer.

[Zip file with NU32\_to\_Chrome Chrome app](http://hades.mech.northwestern.edu/index.php/NU32:_UART_Asynchronous_Serial_Communication)


## Chrome Code ##
main.js
```
chrome.app.runtime.onLaunched.addListener(function() {
  chrome.app.window.create('NU32_to_Chrome.html', {
    id: 'NU32_to_Chrome',
    bounds: {
      width: 850,
      height: 625,
    },
	"resizable": false,
  });
})
```

manifest.json
```
{
  "name": "NU32 to Chrome",
  "version": "1",
  "manifest_version": 2,
  "minimum_chrome_version": "23",
  "description": "Open a serial port, draw a circle with the data",

  "app": {
    "background": {
      "scripts": [ "main.js" ]
    }
  },
  
  "icons": { "128": "serial-128.png" },

  "permissions": [
    "serial"
  ]
}
```

NU32\_to\_Chrome.html
```
<!DOCTYPE html>
<html>
<head>

</head>
<body bgcolor="#6E6E6E">
  <table border=1>
    <tr>
	  <td>
	    Debug Port:
        <select id="debug-port-picker"></select>
	    RX from serial:<p>
	    <textarea id="rx" rows="20" cols="35" readonly style="resize: none;" placeholder="RX Text shows up here!"></textarea>
		<textarea id="log" rows="10" cols="35" readonly style="resize: none;" placeholder="Log messages show up here!"></textarea>
	  </td>
	  <td>
	    <button id="send-T">Toggle LED</button>
	    <canvas id="drawingCanvas" width="512" height="512" style="border:1px solid #000000;"></canvas>
	  </td>
	</tr>
  </table>
  <script src="NU32_to_Chrome.js"></script>
</body>
</html>
```

NU32\_to\_Chrome.js
```
var debugconnectionId = -1;
var RXreadBuffer = "";

function checkSerialClose(){
	if (debugconnectionId != -1) {
      chrome.serial.close(debugconnectionId, ondebugClose);
	  log('Closed port ' + debugconnectionId);
      //return;
    }
};

onload = function() {
  chrome.serial.getPorts(function(ports) {
    buildPortPicker(ports);
  });
};

function buildPortPicker(ports) {
  var eligiblePorts = ports.filter(function(port) {
    return !port.match(/\/dev\/tty/) && port.match(/COM/);
  });
 
  var debugportPicker = document.getElementById('debug-port-picker');
  eligiblePorts.forEach(function(port) {
    var debugportOption = document.createElement('option');
    debugportOption.value = debugportOption.innerText = port;
    debugportPicker.appendChild(debugportOption);
  });

  debugportPicker.onchange = function() {
    if (debugconnectionId != -1) {
      chrome.serial.close(debugconnectionId, ondebugClose);
	  log('Closed port ' + debugconnectionId);
      //return;
    }
    openSelectedDebugPort();
  };
}

function openSelectedDebugPort() {
  var portPicker = document.getElementById('debug-port-picker');
  var selectedPort = portPicker.options[portPicker.selectedIndex].value;
  chrome.serial.open(selectedPort, {bitrate: 230400}, onOpenDebug);
  log('Opened debug port ' + selectedPort);
}

function log(message) {
  document.querySelector('#log').textContent += message + '\n';
  var textarea = document.querySelector('#log');
   if(textarea.selectionStart == textarea.selectionEnd) {
      textarea.scrollTop = textarea.scrollHeight;
   }
};

function logRX(message) {
  document.querySelector('#rx').textContent += message + '\n';
  var textarea = document.querySelector('#rx');
   if(textarea.selectionStart == textarea.selectionEnd) {
      textarea.scrollTop = textarea.scrollHeight;
   }
};

document.querySelector('#send-T').onclick = function() {
  log('Sending T');
  writeSerial("T",debugconnectionId);
};

function onOpenDebug(openInfo) {
  debugconnectionId = openInfo.connectionId;
  log("debugconnectionId: " + debugconnectionId);
  if (debugconnectionId == -1) {
    log('Could not open');
    return;
  }
  else {
    log('Connected');
  
    chrome.serial.setControlSignals(debugconnectionId,{rts:true},function(){});

    chrome.serial.read(debugconnectionId, 1, ondebugRead);
  
    log('Control signals set and reading');
  }
};

function ondebugClose(closeInfo) {
  log("debugconnectionId: " + debugconnectionId);
  debugconnectionId = -1;
  log("Disconnected");
};

function ondebugRead(readInfo) {
  var uint8View = new Uint8Array(readInfo.data);
  var value = String.fromCharCode(uint8View[0]);
    
  if (value == "\n")
  {
      logRX(RXreadBuffer);
	  
	  // get the vlaues from the string
	  var values = RXreadBuffer.split(" ");
	  if (values.length == 3) {
		  //var user = (values[0].trim()).search("1");
		  var user = Number(values[0]);
		  var centerX = Number(values[1])/2;
		  var centerY = Number(values[2])/2;
		  
		  // draw a circle on the canvas
		  var canvas = document.getElementById('drawingCanvas');
		  canvas.width = canvas.width; // clear the canvas
		  var context = canvas.getContext('2d');
		  var radius = 5;

		  context.beginPath();
		  context.arc(centerX, centerY, radius, 0, 2 * Math.PI, false);
		  if (user == 1) {
			context.fillStyle = 'blue';
		  }
		  else {
			context.fillStyle = 'red';
		  }
		  context.fill();
		  context.lineWidth = 1;
		  context.strokeStyle = '#003300';
		  context.stroke();
	  }
	  
      RXreadBuffer = "";
  }
  else
  {
    if(value.charCodeAt() > 0) {
		RXreadBuffer = RXreadBuffer.concat(value);
	}
  }
  // Keep on reading.
  chrome.serial.read(debugconnectionId, 1, ondebugRead);
};

var writeSerial=function(str,conn) {
  chrome.serial.write(conn, str2ab(str), function(){});
};

// Convert string to ArrayBuffer
var str2ab=function(str) {
  var buf=new ArrayBuffer(str.length);
  var bufView=new Uint8Array(buf);
  for (var i=0; i<str.length; i++) {
    bufView[i]=str.charCodeAt(i);
  }
  return buf;
};

```

serial-128.png

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