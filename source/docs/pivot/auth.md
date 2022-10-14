# Use of alternate authentication material

## NTLM authentication flow

| ![NTLM Auth Flow](../../_static/images/ntlm-auth.png) |
|:--:|
| This is for domain authentication. In local authentication, this process only occurs <br>between the client and server, as the server keeps the user's NTLM hash in the SAM |

If an attacker manages to compromise a machine where a domain user is logged in, the attacker may be able to dump 
the domain user's NTLM hash from memory by using a tool like `mimikatz` or other methods. The attacker could try to 
crack the hash(es) and user passwords. 

Another option is "Pass-the Hash":

* User asks to authenticate
* Server sends challenge
* User sends hash (not password)

This would allow an attacker to authenticate as a user in certain situations without ever needing to know the a 
password. It does require dumping hashes locally or remotely.

## Kerberos authentication flow

Ticket Granting Ticket:

| ![Kerberos TGT Auth Flow](../../_static/images/kerberos-tgt.png) |
|:--:|
| As long as the session has not lapsed, the user can reuse the TGT as often as needed <br>to request a TGS. |

Ticket Granting Service:

| ![Kerberos TGS Auth Flow](../../_static/images/kerberos-tgs.png) |
|:--:|
| The TGS also has a service session key, and when the SP decrypts the ticket, the SP <br>will have a session key for the user. |

User Authentication:

| ![Kerberos TGS Auth Flow](../../_static/images/kerberos-user.png) |
|:--:|
| Deny/Allow |

Pass-the-Ticket requires both the ticket and the service session key in order to pass a TGS to a service principal 
to authenticate as a user. A TGT (Golden ticket) allows an attacker to request multiple TGSs (Silver tickets) on 
behalf of a user.

* When a user requests a TGS, they send an encrypted timestamp derived from their password. The algorithm used to 
create this key can be DES (disabled by default on newer Windows installations), RC4, AES218, or AES256, and can 
perhaps be extracted using `mimikatz`. If any of these keys are available on the host, then we can try to request a TGT 
as the user the `Pass-the-Key` way.
* The RC4 hash is equal to a user's NTLM hash. If a users' NTLM hashes were dumped from LSASS during enumeration on a 
domain-joined host, and RC4 a valid encryption algorithm, then these are RC4 hashes, which could be used to request a 
TGT the `Overpass-the-Hash` way.



