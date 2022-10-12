# Setup

Download the vpn config file from THM, and use it to configure 

    sudo openvpn --config ./<username>-lateralmovementandpivoting.ovpn --daemon

Check PID (you get a number, if not, doesn't work):

    sudo ps aux | grep -v grep | grep -i <username>-lateralmovementandpivoting | awk -v FS=' ' '{print $2}'

Set your DNS IPv4 to the IP address of the THMDC in the network diagram, and run:

    sudo systemctl restart NetworkManager

Test:

    nslookup thmdc.za.tryhackme.com

Go to `http://distributor.za.tryhackme.com/creds` and request your credentials for SSH access to `thmjmp2`.

Login with `ssh`:

    $ ssh username@za.tryhackme.com@thmjmp2.za.tryhackme.com 
