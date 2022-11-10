# Credential access

Credential access is where adversaries may find credentials in compromised systems and gain access to user 
credentials. It helps adversaries to reuse them or impersonate the identity of a user. This is an important step 
for lateral movement and accessing other resources such as other applications or systems. Obtaining legitimate 
user credentials is preferred rather than exploiting systems using CVEs.

Credentials are stored insecurely in various locations in systems:

* [Unsecured Credentials: Credentials In Files](https://attack.mitre.org/techniques/T1552/001/) - search a compromised 
machine for credentials in local or remote file systems. Clear-text files could include sensitive information created 
by a user, containing passwords, private keys, etc.
* Database files
* Memory
* Password managers
* Enterprise Vaults
* Active Directory
* Network Sniffing

## PowerShell history

As an example of a history command, a PowerShell saves executed PowerShell commands in a history file in a user profile 
in the following path: `C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.

It might be worth checking what users are working on or finding sensitive information. 

Another example would be finding interesting information. For example, to look for the `password` keyword in the 
Window registry:

    c:\Users\user> reg query HKLM /f password /t REG_SZ /s

OR

    C:\Users\user> reg query HKCU /f password /t REG_SZ /s

## Database Files

Applications use database files to read or write settings, configurations, or credentials. Database files are usually 
stored locally in Windows operating systems. These files are an excellent target to check and hunt for credentials. 
[Configuration files](../breach/config.md) contains a showcase example of extracting credentials from 
the local McAfee Endpoint database file.

## Password Managers

A password manager is an application to store and manage users' login information for local and Internet websites 
and services. Since it deals with users' data, it must be stored securely to prevent unauthorized access. 

Password Manager applications can be built-in (Windows) or third-party, like KeePass, 1Password, LastPass

Misconfiguration and security flaws are found in these applications that let adversaries access stored data. Various 
tools can be used during the enumeration stage to get sensitive data in password manager applications used by 
Internet browsers and desktop applications. 

## Memory Dump

The Operating system's memory is a rich source of sensitive information that belongs to the Windows OS, users, and 
other applications. Data gets loaded into memory at run time or during the execution. Thus, accessing memory is 
limited to administrator users who fully control the system.

The following are examples of memory stored sensitive data, including clear-text credentials, cached passwords, and 
AD Tickets.

## Active Directory

Active Directory stores a lot of information related to users, groups, computers, etc. Thus, enumerating the 
Active Directory environment is one of the focuses of red team assessments. Active Directory has a solid design, 
but misconfiguration made by admins makes it vulnerable to various attacks shown in this room.

The following are some Active Directory misconfigurations that may leak users' credentials:

* Users' description: Administrators set a password in the description for new employees and leave it there, which 
makes the account vulnerable to unauthorised access. 
* Group Policy SYSVOL: Leaked encryption keys let attackers access administrator accounts. 
* NTDS: Contains AD users' credentials, making it a target for attackers.
* AD Attacks: Misconfiguration makes AD vulnerable to various attacks.

## Network Sniffing

Gaining initial access to a target network enables attackers to attack local computers, including the AD environment. 
The Man-In-the-Middle attack against network protocols lets the attacker create a rogue or spoof trusted resources 
within the network to steal authentication information such as NTLM hashes.

## Resources

* [MITRE ATT&CK framework TA0006](https://attack.mitre.org/tactics/TA0006/)
