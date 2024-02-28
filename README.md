License
-------
This script and related content is licensed under the MIT license.
~~~
Copyright 2014 Matthijs Kooijman <matthijs@stdin.nl>
Copyright 2024 Stephan Enderlein (modified and fixed)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
~~~

# Introduction
-------------
This python script allows to monitor the modbus RTU messages with wireshark. \
It connects to a serial port where the modbus usb adapter (RS485) is connected to and creates a file pipe.\
All captured data are put into pcap packages that can be received via this pipe
by whireshark to display the modbus packages.

# Prerequisites
The python script needs following
- python3
- pyserial ```pip install pyserial```

**Steps** (keep order):
- start script
~~~sh
./serial-pcap -b 19200 --fifo /tmp/wireshark /dev/ttyUSB0
~~~
- prepare wireshark to use pipe
- start monitoring pipe

## Start the script
The script connects the usb modbus RS485 adapter and creates a pipe (fifo). \
It waits for the connection of wireshark (or any other program that opens this pipe).
~~~sh
./serial-pcap -b 19200 --fifo /tmp/wireshark /dev/ttyUSB0
~~~

## Prepare Wireshark
Wireshark allows to hear on pipes. It handles the data same way as it
loads a pcap file. The script first creates a pcap header and all other
request and response packages are packed into their own pcap records.

To let wireshark get the needed pcap header, it must be configured and started
**BEFORE** starting the serial-pcap script.

- start wireshark
- configure wireshark to process the user DLT (used by script in pcap header) as modbus rtu packages
  - go to ```Edit->Preferences->Protocol->DLT_USER```
  - edit *Encapsulations Table*
  - add a new entry (if not already) select ```DLT=147``` for DLT and set *Payload protocol* to ```mbrtu```
  - press ok and close preferences
- go to ```Capture->Options->Manage Interfaces->Pipes```, add the pipe ```/tmp/wireshark``` and press ok
- select the pipe (in current dialog, which is added to the list)
- press "start"

When there was a pipe already created before (manually via ```mkfifo /tmp/wireshark``` or by a previous call of serial-pcap) wireshark starts monitoring

Unfortunately wireshark only remembers the Encapsulations Tabel entries, but
not the pipe. This must be configured each time after starting wireshark


# Changes
-------
Project is originally cloned from https://github.com/Pinoccio/tool-serial-pcap

- use a user DLT (data link type) and remove notworking pcap encapsulation
- remove blocking code that prevents getting data from usb modbus adapter
- keep serial port always open instead of open/close on each cycle (avoids loosing data)


# Links
- https://datatracker.ietf.org/doc/id/draft-gharris-opsawg-pcap-00.html
- https://www.tcpdump.org/linktypes.html
