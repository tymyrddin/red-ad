# Moving through the network

* Local administrator accounts may be repeated across multiple hosts on the network. Even if that's the case a 
local administrator cannot access a computer remotely with admin privileges using WinRM, SMB, or RPC. The local 
administrator must use RDP to open an administrative session on a host. This setting can be changed.
* The built-in default administrator account is not subject to UAC, while other local administrator accounts are.
* Domain accounts with local admin can open an administrative login using RDP, WinRM, SMB, or RPC. This can be disabled.