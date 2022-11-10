# Local Windows credentials

Keylogger is a software or hardware device to monitor and log keyboard typing activities. Keyloggers were initially 
designed for legitimate purposes such as feedback for software development or parental control. And, they can be 
misused to steal data. As a red teamer, hunting for credentials through keyloggers in a busy and interactive 
environment is a good option. If we know a compromised target has a logged-in user, we can use keylogging with tools 
like the Metasploit framework. An example use case can be found in [Exploiting AD users](../exploit/users.md). 

## Security Account Manager (SAM)

The SAM is a Microsoft Windows database that contains local account information such as usernames and passwords. The 
SAM database stores these details in an encrypted format to make them harder to be retrieved. It can not be read and 
accessed by any users while the Windows operating system is running. And there are ways and attacks to dump the 
content of the SAM database. 

* The first method is using the built-in Metasploit Framework feature, hashdump, to get a copy of the content of the 
SAM database. The Metasploit framework uses in-memory code injection to the `LSASS.exe` process to dump copy hashes.
* Another approach uses the 
[Microsoft Volume shadow copy service](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service), 
which helps perform a volume backup while applications read/write on volumes.
* Another possible method for dumping the SAM database content is through the Windows Registry. Windows registry 
also stores a copy of some SAM database contents to be used by Windows services.

### Shadow Copy

Use `wmic` (with administrator privileges) to create a shadow volume copy:

1. Run the standard cmd.exe prompt with administrator privileges.
2. Execute the wmic command to create a copy shadow of `C:` drive
3. Verify the creation from step 2 is available.
4. Copy the SAM database from the volume created in step 2

Copy shadow of `C:` drive:

    C:\Users\Administrator>wmic shadowcopy call create Volume='C:\'

To list and confirm that we have a shadow copy of the `C:` volume:

    C:\Users\Administrator>vssadmin list shadows

A shadow copy volume was created: `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1`

The SAM database is encrypted either with RC4 or AES encryption algorithms. In order to decrypt it, we need the 
decryption key stored in `c:\Windows\System32\Config\system`. 

Copy both files (sam and system) from the shadow copy volume to the desktop, and then to the attack machine.

### Registry Hives

Save the value of the Windows registry using the reg.exe tool (with Administrator privileges):

    C:\Users\Administrator\Desktop>reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg
    C:\Users\Administrator\Desktop>reg save HKLM\system C:\users\Administrator\Desktop\system-reg

Decrypt it using secretsdump.py:

    # python3 /opt/impacket/examples/secretsdump.py -sam /tmp/sam-reg -system /tmp/system-reg LOCAL