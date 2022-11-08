# Persistence through ACLs

Active Directory has a process called `SDProp` that replicates a template called `AdminSDHolder` to all protected 
groups in the domain. If an attacker adds their user account to the AdminSDHolder template, `SDProp` will replicate 
the ACL to all the protected groups when it runs every 60 minutes. So even if the attacker is removed from 
privileged groups, they will be re-added at very cycle by `SDProp`.

`RDP` to `thmwrk1` with the unprivileged user account:

    xfreerdp /v:thmwrk1.za.tryhackme.loc /u:'user.name' /p:'Password'

Inject the network credentials of the domain administrator into a session:

    runas /netonly /user:za.tryhackme.loc\Administrator cmd.exe

## Modify the AdminSDHolder template

1. From the command prompt running with the injected domain admin credential, run `mmc.exe`. 
2. Go to File -> Add/Remove Snap-in
3. Add the Active Directory Users and Computers snap-in.
4. Add the unprivileged user to the ACL here and allow Full Control for the user. 
5. Manually start the SDProp sync procedure. 

## WinRM to the Domain Controller

In the injected session, enter a PowerShell session:

    powershell -ep bypass

WinRM to the domain controller as the DA:

    Enter-PSSession -ComputerName thmdc.za.tryhackme.loc

Now running a PowerShell session on the domain controller:

    Import-Module C:\Tools\Invoke-ADSDPropagation.ps1
    Invoke-ADSDPropagation

