# Setup for THM AD

## Connecting to the network

Download the vpn config file from THM, and use it to configure 

    sudo openvpn --config ./<some assigned code>-persistingad.ovpn --daemon

Check PID (you get a number, if not, does not work):

    sudo ps aux | grep -v grep | grep -i <some assigned code>-persistingad.ovpn | awk -v FS=' ' '{print $2}'

## Edit DNS configuration

Set your DNS IPv4 to the IP address of the THMCHILDDC in the network diagram (also add 1.1.1.1 for connections to 
the internet) and run:

    sudo systemctl restart NetworkManager

## Test hostname lookups

```text
$ dig thmdc.za.tryhackme.loc

; <<>> DiG 9.18.7-1-Debian <<>> thmdc.za.tryhackme.loc
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 468
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;thmdc.za.tryhackme.loc.		IN	A

;; ANSWER SECTION:
thmdc.za.tryhackme.loc.	1200	IN	A	10.200.60.101

;; Query time: 40 msec
;; SERVER: 10.200.60.101#53(10.200.60.101) (UDP)
;; WHEN: Mon Nov 07 14:15:34 GMT 2022
;; MSG SIZE  rcvd: 67
```

And:

```text
$ nslookup google.com
;; communications error to 10.200.60.101#53: timed out
;; communications error to 10.200.60.101#53: timed out
;; communications error to 10.200.60.101#53: timed out
Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
Name:	google.com
Address: 216.58.214.78
;; communications error to 10.200.60.101#53: timed out
;; communications error to 10.200.60.101#53: timed out
;; communications error to 10.200.60.101#53: timed out
Name:	google.com
Address: 2a00:1450:4007:818::200e
```

## Request credentials

Get your credentials from `http://distributor.za.tryhackme.loc/creds`.

## Jump in

You can now either login with `ssh`, for example:

    $ ssh za.tryhackme.loc\\<AD Username>@thmwrk1.za.tryhackme.loc

Or by RDP:

    $ xfreerdp /v:thmwrk1.za.tryhackme.loc /u:'<AD Username>' /p:'<AD Password>'
