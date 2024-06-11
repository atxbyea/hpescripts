
This is a nice convenient way to gather MAC-addresses from a greenfield deployment for remote configuration \ OS rollout of hosts


Scenario :

20 racks of servers with ToR Management \ OOB network connected, no existing DHCP server

Let's install a DHCP server on our laptop \ machine, I run it as a systemd service attached to a specific adapter, you can whatever you prefer, for example docker.

Connect the machine to the ToR OOB network then start the DHCP server, after a few minutes check how many clients there are.

Since iLOs by default have `ILOSERIALNUMBER` as the hostname you can grep for the iLO clients directly.

Example with isc-dhcp

`dhcp-lease-list | grep ILO | wc -l`

When you are satisfied, write it to a file

`dhcp-lease-list | grep ILO > leases.txt`

Now prune said list

`cat leases.txt | cut -d . -f 4 | cut -d " " -f1 > iloip.txt`

We now have a list we can use to script the info we need from the server, in this case the MAC addresses of the Network Cards and the iLOs, to verify a server has been dumped, lets also light the UID.

```
for i in $(cat iloip.txt)
   do ilorest list --select EthernetInterface.v1_4_1 --url https://192.168.1.$i -u Administrator -p PaSsWoRd | grep -E 'MAC|FQDN'
      ilorest set LocationIndicatorActive=true 
      ilorest commit &
   done > MAC-addresses
```

Example output of the file will look similar to this 

```
MACAddress=11:22:33:44:55:ed
MACAddress=11:22:33:44:55:ef
FQDN=None
MACAddress=11:22:33:44:55:47
PermanentMACAddress=11:22:33:44:55:47
MACAddress=11:22:33:44:55:04
PermanentMACAddress=11:22:33:44:55:04
MACAddress=11:22:33:44:55:31 <---|
MACAddress=11:22:33:44:55:ec <---| If the server has a 4 port LOM look for the lowest digit for the first port (remember this is hexadecimal, so 9 is lower than a etc)
MACAddress=11:22:33:44:55:ee <---|
FQDN=ILOCZ123456AB.eirikz.dhcp <--- Hostname and Serial Number
MACAddress=11:22:33:44:55:46
PermanentMACAddress=11:22:33:44:55:46 <--- One of the permanent ones is iLO, look for the one that is not in a series
```

This script is not fully paralell, and has room for improvements, but it works fine, takes about 30m for 300 servers, most of that time is the wait time to get confirmation from ilorest.
