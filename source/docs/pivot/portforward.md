# Port forwarding

## SSH tunnelling

SSH, as it already has built-in functionality to do port forwarding through a feature called SSH Tunneling. While 
SSH used to be a protocol associated with Linux systems, Windows now ships with the OpenSSH client by default, so 
you can expect to find it in many systems nowadays, independent of their operating system.

On Windows machines, most likely no SSH server will be available: 

* Start a tunnel from the compromised machine, acting as an `ssh` client, to the attack machine, which will act as an 
`ssh` server. 
* Making a connection back to our attacker's machine, we will want to create a user in it without access to any 
console for tunnelling and set a password to use for creating the tunnels.
* The SSH tunnel can be used to do either local or remote port forwarding.

```text
useradd tunneluser -m -d /home/tunneluser -s /bin/true
passwd tunneluser
```

### SSH remote port forwarding

Remote port forwarding allows you to take a reachable port from the SSH client (the compromised machine) and project 
it into a remote ssh server (the attacker's machine).

* Open a port in the attacker's machine that can be used to connect back to port 3389 in the server through 
the SSH tunnel

### SSH local port forwarding

## Port forwarding with socat

## Dynamic port forwarding and SOCKS

