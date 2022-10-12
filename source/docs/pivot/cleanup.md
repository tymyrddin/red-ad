# Cleanup

To stop the vpn daemon started in [setup](setup.md), get PID:

    sudo ps aux | grep -v grep | grep -i <username>-lateralmovementandpivoting | awk -v FS=' ' '{print $2}'
    <PID>

and kill the process:

    sudo kill -9 <PID>

Remove the IP address of the THMDC in DNS IPv4 setting and run:

    sudo systemctl restart NetworkManager

Done.