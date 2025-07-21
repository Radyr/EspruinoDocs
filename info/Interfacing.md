<!--- Copyright (c) 2013 Gordon Williams, Pur3 Ltd. See the file LICENSE for copying permission. -->
Interfacing to a PC
=================

<span style="color:red">:warning: **Please view the correctly rendered version of this page at https://www.espruino.com/Interfacing. Links, lists, videos, search, and other features will not work correctly when viewed on GitHub** :warning:</span>

* KEYWORDS: Interfacing,PC,Computer,Connect,Control,USB,Bluetooth,Serial,CDC,BLE,Bluetooth LE,gatttool,hcitool,Noble,node-ble,noble-winrt
* USES: BLE,USB,Web Bluetooth

You can use Espruino directly from your PC, Mac or Raspberry Pi to turn things on and off or measure values.

Espruino devices appear as a serial port, and on that serial port they present a
REPL (the console). Normally you would connect with a VT100-compatible terminal
and you can write code, however you can send commands straight to them as if
you were typing the command directly at the REPL.

**NOTE:** Because the devices expect to be outputting to a VT100 terminal, they
will return extra characters - 'echoing' what was written back to the terminal.
If you're sending a lot of commands you can either turn off echo permanently
by sending the command `"echo(0)\n"`, or you can turn it off just for a line
by sending character code 16 `"\x10"` as the first character on the line - eg. `"\x10LED.toggle()\n"`.

* APPEND_TOC

USB / Serial
------------

### Web Serial (on Web Pages)

You can use the [`Web Serial API`](https://codelabs.developers.google.com/codelabs/web-serial/#0) to access
devices from a Web Page.

To make this easier we've made the [UART.js](/UART.js) Library to provide a consistent API for accessing Serial/Bluetooth devices from the web:

```HTML_demo_link
<html>
 <head>
 </head>
 <body>
  <script src="https://www.espruino.com/js/uart.js"></script>
  <button onclick="UART.write('LED1.set();\n');">LED On!</button>
  <button onclick="UART.write('LED1.reset();\n');">LED Off!</button>
 </body>
</html>
```


### Windows

You can write to Espruino very easily with the Windows Command Prompt. For instance to turn an LED on, the command is:

```Batchfile
echo LED1.set() > \\.\COM10
```

Where COM10 is the COM port of your device. If you want to wrap this up in a shortcut to go on the desktop, just enter the following as the shortcut location:

```Batchfile
cmd.exe /c "echo LED1.set() > \\.\COM10"
```

If you're not connecting by USB, you may have to set up the baud rate first. You have to do this with:

```Batchfile
MODE COM10:9600,N,8,1
```

### Mac, Linux or Raspberry Pi

This is very similar to windows as long as you know the device name of Espruino.

  **Note:** On Linux, devices will be `/dev/ttyACM0`, `ttyAMA0`, etc, but on MacOS you'll want to use the `/dev/cu.usmodem1234` device name.

```Bash
echo "LED1.set()" > /dev/ttyACM0
```

If you're not connecting by USB, you may have to set up the baud rate first. You have to do this with:

```Bash
stty -F /dev/ttyACM0 9600
```

### Python (Multiplatform)

There's a [good thread on it here](http://www.raspberrypi.org/phpBB3/viewtopic.php?f=45&t=19301&p=188619)

However, it's very simple:

* In your favourite language, open the serial port at 9600bps.
* Send the text ```echo(0)``` (and a newline) - this turns off echoing, which means that the only text Espruino sends is that which comes from ```print(...)```.
* Send javascript commands, like ```digitalWrite(LED1,1)```
* Or read back values by sending commands like ```print(analogRead(A0))```, and waiting a fraction of a second for the result to appear.
* When you exit, send the text ```echo(1)``` (and a newline) - this will turn echoing back on so that next time you connect with a terminal, Espruino responds to your keypresses in the way you'd expect.

There's some example Python code here:

```Python
#!/usr/bin/python
import time
import serial
import sys
import json

def espruino_cmd(command):
 ser = serial.Serial(
  port='/dev/ttyACM0', # or /dev/ttyAMA0 for serial on the Raspberry Pi
  baudrate=9600,
  parity=serial.PARITY_NONE,
  stopbits=serial.STOPBITS_ONE,
  bytesize=serial.EIGHTBITS,
  xonxoff=0, rtscts=0, dsrdtr=0,
 )
 ser.isOpen()
 ser.write(command+"\n")
 endtime = time.time()+0.2 # wait 0.2 sec
 result = ""
 while time.time() < endtime:
  while ser.inWaiting() > 0:
   result=result+ser.read(1)
 ser.close()
 return result

# Read 1 analog
#print espruino_cmd("print(analogRead(A1))").strip()
# Read 3 analogs into an array
#print espruino_cmd("print([analogRead(A1),analogRead(A2),analogRead(A3)])").strip().split(',')

if len(sys.argv)!=2:
 print "USAGE: espruino_command.py "+'"'+"print('hello')"+'"'
 exit(1)

print espruino_cmd(sys.argv[1]).strip()
```

You can just run this from the shell with commands like:

```Bash
$ ./espruino_command.py "echo(0)"
echo(0)
$ ./espruino_command.py "print('hello')"
hello
$ ./espruino_command.py "print(analogRead(A0))"
0.6546
$ ./espruino_command.py "digitalWrite(LED1,1)"
[no output, but turns the LED on]
```

To be able to communicate with the serial port using Python, you will need pySerial: [http://pyserial.sourceforge.net](http://pyserial.sourceforge.net) . If you run into the trouble that the code complains that it cannot find `serial`, you will first need to install pySerial, which is luckily very straightforward (example below assumes you have downloaded version 2.7, update the commands as appropriate):

```Bash
tar xfvz pyserial-2.7.tar.gz
cd pyserial-2.7
sudo python setup.py install
```

Bluetooth
---------

**For Bluetooth LE as used on Puck.js/Pixl.js/MDBT42Q, see the next heading**

You can add an HC-05 or HC-06 [[Bluetooth]] module to the back of an Espruino [[Original]],
or can wire one up to the default UART pins of any Espruino board in order to
communicate with the board wirelessly.

In this case, the Bluetooth Module behaves as a wireless serial port. Once
you pair with your Operating System, a new Serial Port device will appear
and you can connect exactly as you would have done for Serial/USB above.


Bluetooth LE
-------------

If you just wish to receive non-private data from an Espruino Bluetooth LE device (eg temperature, or
when a button is pressed) we'd recommend that you use [Bluetooth LE Advertising](/BLE+Advertising). This
is connectionless and low power, so is flexible and robust. [See this link](/BLE+Advertising) for
code examples.

For two way communication, Espruino Bluetooth LE devices provide a serial port as something called the
'Nordic UART Service'. While this is now used by many devices, it is
not part of the Bluetooth SIG's standard for Bluetooth LE, so no operating
system will add it as a communications port in the same way that USB devices
get added.

As a result you have to access the Nordic UART service via Bluetooth LE directly.

### Web Bluetooth (on Web Pages)

Check out the page on [using Espruino with Web Bluetooth](http://www.espruino.com/Web+Bluetooth).

```HTML_demo_link
<html>
 <head>
 </head>
 <body>
  <script src="https://www.puck-js.com/puck.js"></script>
  <button onclick="Puck.write('LED1.set();\n');">LED On!</button>
  <button onclick="Puck.write('LED1.reset();\n');">LED Off!</button>
 </body>
</html>
```

We've also made the [UART.js](/UART.js) Library to provide a consistent API for accessing Serial/Bluetooth devices from the web.

### Node.js / JavaScript

On Node.js, [`noble`](https://www.npmjs.com/package/noble) has be the accepted way of getting Bluetooth LE
access for years. **However, it is now unmaintained.** As a result, there are other versions that you should use instead
that are code-compatible but work on different platforms:

* **Mac OS Mojave** has [broken BLE support in noble](https://github.com/noble/noble/issues/834) so you may want to use the [noble-mac](https://github.com/Timeular/noble-mac) library instead.
* **Windows 10/11** `noble` needs direct access to the bluetooth device on Windows, which is [difficult to accomplish](https://github.com/noble/noble#windows) and not recommended, so
[`noble-winrt`](https://www.npmjs.com/package/noble-winrt) or [`noble-uwp`](https://www.npmjs.com/package/noble-uwp) use Windows' own bluetooth driver which works a lot better.
  * If you do need to use the original `noble` on Windows for some reason, it's possible to install an extra [USB Bluetooth LE dongle](http://www.espruino.com/Puck.js+Quick+Start#requirements)**
  that Windows doesn't have a Bluetooth driver installed for and configure it with [Zadig](https://zadig.akeo.ie/). This will allow you to keep you existing Bluetooth working.
* **Linux** [`@abandonware/noble`](https://www.npmjs.com/package/@abandonware/noble) is a version of `noble` with some fixes applied, and it normally works ok on Linux once you run the `setcap` command (see code below).
  * [node-ble](https://www.npmjs.com/package/node-ble) is another option which uses DBUS to access Bluetooth so works alongside the OS rather than trying
  to use Bluetooth directly. The API is different though - see https://github.com/espruino/EspruinoTools/blob/master/core/serial_node_ble.js for usage.

Assuming we're on Linux you can run `npm install @abandonware/noble`, then use this code, otherwise you'll have to change the `require(...)` line to pull in the correct module:

```JS
/* On Linux, BLE normally needs admin right to be able to access BLE
 *
 * sudo setcap cap_net_raw+eip $(eval readlink -f `which node`)
 */


var noble = require('@abandonware/noble'); // Linux
// var noble = require('noble-winrt'); // Windows
// var noble = require('noble-mac'); // Mac OS

var ADDRESS = "ff:a0:c7:07:8c:29";
var COMMAND = "\x03\x10clearInterval()\n\x10setInterval(function() {LED.toggle()}, 500);\n\x10print('Hello World')\n";

var btDevice;
var txCharacteristic;
var rxCharacteristic;

noble.on('stateChange', function(state) {
 console.log("Noble: stateChange -> "+state);
  if (state=="poweredOn")
    noble.startScanning([], true);
});

var foundDevice = false;
noble.on('discover', function(dev) {
  if (foundDevice) return;
  console.log("Found device: ",dev.address);
  if (dev.address != ADDRESS) return;
  noble.stopScanning();
  // noble doesn't stop right after stopScanning is called,
  // so we have to use foundDevice to ensure we onloy connect once
  foundDevice = true;
  // Now connect!
  connect(dev, function() {
    // Connected!
    write(COMMAND, function() {
      btDevice.disconnect();
    });
  });
});

function connect(dev, callback) {
  btDevice = dev;
  console.log("BT> Connecting");
  btDevice.on('disconnect', function() {
    console.log("Disconnected");
  });
  btDevice.connect(function (error) {
    if (error) {
      console.log("BT> ERROR Connecting",error);
      btDevice = undefined;
      return;
    }
    console.log("BT> Connected");
    btDevice.discoverAllServicesAndCharacteristics(function(error, services, characteristics) {
      function findByUUID(list, uuid) {
        for (var i=0;i<list.length;i++)
          if (list[i].uuid==uuid) return list[i];
        return undefined;
      }

      var btUARTService = findByUUID(services, "6e400001b5a3f393e0a9e50e24dcca9e");
      txCharacteristic = findByUUID(characteristics, "6e400002b5a3f393e0a9e50e24dcca9e");
      rxCharacteristic = findByUUID(characteristics, "6e400003b5a3f393e0a9e50e24dcca9e");
      if (error || !btUARTService || !txCharacteristic || !rxCharacteristic) {
        console.log("BT> ERROR getting services/characteristics");
        console.log("Service "+btUARTService);
        console.log("TX "+txCharacteristic);
        console.log("RX "+rxCharacteristic);
        btDevice.disconnect();
        txCharacteristic = undefined;
        rxCharacteristic = undefined;
        btDevice = undefined;
        return openCallback();
      }

      rxCharacteristic.on('data', function (data) {
        var s = "";
        for (var i=0;i<data.length;i++) s+=String.fromCharCode(data[i]);
        console.log("Received", JSON.stringify(s));
      });
      rxCharacteristic.subscribe(function() {
        callback();
      });
    });
  });
};

function write(data, callback) {
  function writeAgain() {
    if (!data.length) return callback();
    var d = data.substr(0,20);
    data = data.substr(20);
    var buf = Buffer.alloc(d.length);
    for (var i = 0; i < buf.length; i++)
      buf.writeUInt8(d.charCodeAt(i), i);
    console.log("BT> Write "+JSON.stringify(buf.toString("binary")));
    txCharacteristic.write(buf, false, writeAgain);
  }
  writeAgain();
}

function disconnect() {
  btDevice.disconnect();
}
```

### Python

There are a few different Bluetooth LE packages for Python.

* [`bluepy`](https://github.com/IanHarvey/bluepy) is older but only supports Linux.
* [`bleak'](https://pypi.org/project/bleak/) is a newer multiplatform Python Library that supports Windows, Mac OS and Linux.

#### Bleak

You need to run `pip install bleak`, and you can then do:

```Python
import asyncio
import array
from bleak import discover
from bleak import BleakClient

address = "dd:0c:e4:29:32:ab"
UUID_NORDIC_TX = "6e400002-b5a3-f393-e0a9-e50e24dcca9e"
UUID_NORDIC_RX = "6e400003-b5a3-f393-e0a9-e50e24dcca9e"
command = b"\x03\x10clearInterval()\n\x10setInterval(function() {LED.toggle()}, 500);\n\x10print('Hello World')\n"

def uart_data_received(sender, data):
    print("RX> {0}".format(data))

# You can scan for devices with:
#async def run():
#    devices = await discover()
#    for d in devices:
#        print(d)

print("Connecting...")
async def run(address, loop):
    async with BleakClient(address, loop=loop) as client:
        print("Connected")
        await client.start_notify(UUID_NORDIC_RX, uart_data_received)
        print("Writing command")
        c=command
        while len(c)>0:
          await client.write_gatt_char(UUID_NORDIC_TX, bytearray(c[0:20]), True)
          c = c[20:]
        print("Waiting for data")
        await asyncio.sleep(1.0, loop=loop) # wait for a response
        print("Done!")


loop = asyncio.get_event_loop()
loop.run_until_complete(run(address, loop))
```

#### Bluepy

[`bluepy`](https://github.com/IanHarvey/bluepy) only supports Linux.

You need to run `pip install bluepy`, and you can then do:

```Python
# USAGE:
# python bluepy_uart.py ff:a0:c7:07:8c:29

import sys
from bluepy import btle
from time import sleep

if len(sys.argv) != 2:
  print "Fatal, must pass device address:", sys.argv[0], "<device address="">"
  quit()

# \x03 -> Ctrl-C clears line
# \x10 -> Echo off for line so don't try and send any text back
#command = "\x03\x10reset()\nLED.toggle()\n"
command = "\x03\x10clearInterval()\n\x10setInterval(function() {LED.toggle()}, 500);\n\x10print('Hello World')\n"

# Handle received data
class NUSRXDelegate(btle.DefaultDelegate):
    def __init__(self):
        btle.DefaultDelegate.__init__(self)
        # ... initialise here
    def handleNotification(self, cHandle, data):
        print('RX: ', data)
# Connect, set up notifications
p = btle.Peripheral(sys.argv[1], "random")
p.setDelegate( NUSRXDelegate() )
nus = p.getServiceByUUID(btle.UUID("6E400001-B5A3-F393-E0A9-E50E24DCCA9E"))
nustx = nus.getCharacteristics(btle.UUID("6E400002-B5A3-F393-E0A9-E50E24DCCA9E"))[0]
nusrx = nus.getCharacteristics(btle.UUID("6E400003-B5A3-F393-E0A9-E50E24DCCA9E"))[0]
nusrxnotifyhandle = nusrx.getHandle() + 1
p.writeCharacteristic(nusrxnotifyhandle, b"\x01\x00", withResponse=True)
# Send data (chunked to 20 bytes)
while len(command)>0:
  nustx.write(command[0:20]);
  command = command[20:];
# wait for data to be received
while p.waitForNotifications(1.0): pass
# No more data for 1 second, disconnect
p.disconnect()
```

### gatttool on Linux / Raspberry Pi shell

It's also possible to use the pre-installed Linux bluetooth tools to write
directly from the shell. You can either send raw JavaScript code to an otherwise
unprogrammed Bluetooth LE Espruino device.

#### Finding the device

Just run `sudo hcitool lescan` and wait until you see the device with the name
you want (here we're interested in `MDBT42Q c774`), then use Ctrl-C to break out.

```sh
$ sudo hcitool lescan
DA:34:7C:4C:5A:47 Puck.js 5a47
DA:34:7C:4C:5A:47 (unknown)
C4:7C:8D:6A:AC:79 (unknown)
C4:7C:8D:6A:AC:79 Flower care
F5:3E:A0:62:C7:74 MDBT42Q c774
F5:3E:A0:62:C7:74 (unknown)
```

Copy the device's address from the terminal, and use it in subsequent commands.

Next we want to convert our JavaScript command into a form (hexadecimal bytes)
that can be used by `gatttool`. We'll send `"LED.toggle()\n"`, so use the following
JavaScript code to convert that to hex bytes:

```JS
"LED.toggle()\n".split("").map(x=>(256+x.charCodeAt()).toString(16).substr(-2)).join("")
// ="4c45442e746f67676c6528290a"
```

(or the following in the shell)

```sh
echo "LED.toggle()\n" | od -A n -t x1 | tr -d " "
```

Now we'll run `gatttool`:

```sh
$ gatttool --addr-type=random -I --device=F5:3E:A0:62:C7:74
# Now connect to the device
[F5:3E:A0:62:C7:74][LE]> connect
Attempting to connect to F5:3E:A0:62:C7:74
Connection successful
# List characteristics - we want
# 6e400002-b5a3-f393-e0a9-e50e24dcca9e which is
# the Nordic TX characteristic
[F5:3E:A0:62:C7:74][LE]> char-desc
handle: 0x0001, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x0002, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0003, uuid: 00002a00-0000-1000-8000-00805f9b34fb
handle: 0x0004, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0005, uuid: 00002a01-0000-1000-8000-00805f9b34fb
handle: 0x0006, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0007, uuid: 00002a04-0000-1000-8000-00805f9b34fb
handle: 0x0008, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0009, uuid: 00002aa6-0000-1000-8000-00805f9b34fb
handle: 0x000a, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x000b, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x000c, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x000d, uuid: 6e400003-b5a3-f393-e0a9-e50e24dcca9e
handle: 0x000e, uuid: 00002902-0000-1000-8000-00805f9b34fb
handle: 0x000f, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0010, uuid: 6e400002-b5a3-f393-e0a9-e50e24dcca9e
# Now we write our command to the handle of that characteristic (0x0010)
# LED1 will turn on
[F5:3E:A0:62:C7:74][LE]> char-write-req 0x0010 4c45442e746f67676c6528290a
Characteristic value was written successfully
# Writing again turns LED1 off
[F5:3E:A0:62:C7:74][LE]> char-write-req 0x0010 4c45442e746f67676c6528290a
Characteristic value was written successfully
[F5:3E:A0:62:C7:74][LE]> quit
```

You can wrap this up into one shell command:

```sh
gatttool --addr-type=random --device=F5:3E:A0:62:C7:74 --char-write-req --handle=0x0010 \
  --value=4c45442e746f67676c6528290a
# or to convert the whole command:
gatttool --addr-type=random --device=F5:3E:A0:62:C7:74 --char-write-req --handle=0x0010 \
 --value=`echo "LED.toggle()\n" | od -A n -t x1 | tr -d " "`
```

This assumes the handle of the UART TX characteristic is always 0x0010, which should
be the case on up to date Espruino firmwares unless you have enabled other services
such as HID.


Custom characteristic
----------------------

It can be a bit messy [and also insecure](/BLE+Security) to leave your
Espruino device entirely open to execute JavaScript, so instead you
can create a characteristic to do what you want, and then write to that:

* Connect with the Web IDE
* Upload this code:

```JS
NRF.setServices({
  "35ac0001-18b0-e8b7-3feb-62cec301da00" : {
    "35ac0002-18b0-e8b7-3feb-62cec301da00" : {
      value : [0],
      maxLen : 1,
      writable : true,
      onWrite : function(evt) {
        digitalWrite([LED2,LED1], evt.data[0]);
        // This could be extended for more data bytes...
      }
    }
  }
});
```

This will create a characteristic with uuid `35ac0002-18b0-e8b7-3feb-62cec301da00`
 (which was randomly generated - we'd recommend you generate your own with `date|md5sum`).
 When written to it'll update the state of the two LEDs based on the bottom 2 bits
 of the data sent to it.

* disconnect the IDE and use gatttool:

```sh
$ gatttool --device=F5:3E:A0:62:C7:74 --addr-type=random -I
[F5:3E:A0:62:C7:74][LE]> connect
Attempting to connect to F5:3E:A0:62:C7:74
Connection successful
[F5:3E:A0:62:C7:74][LE]> char-desc
handle: 0x0001, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x0002, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0003, uuid: 00002a00-0000-1000-8000-00805f9b34fb
handle: 0x0004, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0005, uuid: 00002a01-0000-1000-8000-00805f9b34fb
handle: 0x0006, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0007, uuid: 00002a04-0000-1000-8000-00805f9b34fb
handle: 0x0008, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0009, uuid: 00002aa6-0000-1000-8000-00805f9b34fb
handle: 0x000a, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x000b, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x000c, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x000d, uuid: 6e400003-b5a3-f393-e0a9-e50e24dcca9e
handle: 0x000e, uuid: 00002902-0000-1000-8000-00805f9b34fb
handle: 0x000f, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0010, uuid: 6e400002-b5a3-f393-e0a9-e50e24dcca9e
handle: 0x0011, uuid: 00002800-0000-1000-8000-00805f9b34fb
handle: 0x0012, uuid: 00002803-0000-1000-8000-00805f9b34fb
handle: 0x0013, uuid: 35ac0002-18b0-e8b7-3feb-62cec301da00
[F5:3E:A0:62:C7:74][LE]> char-write-req 0x0013 01
# <----------- LED1 turns on
Characteristic value was written successfully
[F5:3E:A0:62:C7:74][LE]> char-write-req 0x0013 02
# <----------- LED2 turns on
Characteristic value was written successfully
[F5:3E:A0:62:C7:74][LE]> char-write-req 0x0013 02
# <----------- both LEDS turns on
Characteristic value was written successfully
[F5:3E:A0:62:C7:74][LE]> char-write-req 0x0013 00
# <----------- both LEDS turns off
Characteristic value was written successfully
[F5:3E:A0:62:C7:74][LE]> quit
```

And again this can be rolled up into one command (assuming your handle is the same).

```sh
gatttool --addr-type=random --device=F5:3E:A0:62:C7:74 --char-write-req --handle=0x0013 --value=01
```
