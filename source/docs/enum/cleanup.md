# Cleanup

Remove the sharphound zip and directory.

Close bloodhound.

Stop neo4j:

    neo4j stop

To stop the vpn daemon started in [setup](setup.md), get PID:

    sudo ps aux | grep -v grep | grep -i <username>-adenumeration.ovpn | awk -v FS=' ' '{print $2}'
    <PID>

and kill the process:

    sudo kill -9 <PID>

Remove the IP address of the THMDC in DNS IPv4 setting and run:

    sudo systemctl restart NetworkManager

Done.