# Through PowerShell

All domain users:

    Get-ADUser -Filter * 

Find any user where name ends in `... phillips`:

    Get-ADUser -Filter 'Name -like "*phillips"'

Find user `beth.nolan` and return all properties:

    Get-ADUser -Identity john.doe -Properties * – 

All domain groups:

    Get-ADGroup -Filter *

Pipe the Administrators group object to Get-ADGroupMember to retrieve members of the group:

    Get-ADGroup -Identity Administrators | Get-ADGroupMember

Get any domain objects that we modified on or after a specific date and time:

    $modifiedDate = Get-Date '2022/10/11'
    Get-ADObject -Filter 'whenChanged -ge $modifiedDate' -IncludeDeletedObjects

Get information about the domain from the domain controller:

    Get-ADDomain

Change a User Password:

    $oldPass = Read-Host -AsSecureString -Prompt 'Enter the old password'
    $newPass = Read-Host -AsSecureString -Prompt 'Enter the new password'
    Set-ADAccountPassword -Identity user.name -OldPassword $oldpPass -NewPassword $newPass
