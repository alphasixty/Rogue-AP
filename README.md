# Rogue-AP
Method to create Rogue AP with Raspberry Pi 3+

Raspberry Pi 3+ Rogue AP
Run updates and check interface compatibility:

>> Using Kali for this as it was already installed on my Pi’s - armv71 

>> Update

```bash

sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade

```bash
>> Install hostapd and dnsmasq

>> Make sure you can put your wireless adapter in monitor mode

```

ifconfig [interface] down
iwconfig [interface] mode monitor

```

##Use ifconfig to verify the adapter mode##

Creating the AP:

>> Create /etc/hostapd.conf

```

touch /etc/hostpad.conf

```

>> Configure hostapd.conf - I used vim for this, doing this will also create the file if it is not there alredy. 

```

sudo vim /etc/hostpad.conf

```

#Sets the interface that will be the acting AP
interface=[interface put in monitor mode]
#Sets the appropriate driver
driver=nl80211
#Set ssid to whatever you want to broadcast
ssid=[fake AP name]
hw_mode=g
channel=[fake AP channel]
macaddr_acl=0
ignore_broadcast_ssid=0
>> Initiate fake AP >> hostapd /etc/hostapd.conf

##Error Troubleshooting##
##Failed to restart hostapd.service:Unit hostapd.service is masked.

```

sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd

```
>> Failed to start Advanced IEEE 802.11…….. Probably because we did not put WPA params in the conf || This had no affect on the initialization of the AP

DHCP:

>> Use dnsmasq to set DHCP

>> Configure dnsmasq.conf in /etc/dnsmasq.conf

#Sets interface to listen for DHCP and DNS requests
interface=[interface in monitor mode]

#Binds DHCP to IP as well as the above interface
listen-address=127.0.0.1

#Sets the network that DHCP will operate on
dhcp-range=192.168.1.2,192.168.1.30,255.255.255.0.12h [this just needs to be a valid network]

#DHCP option 3 sets the gateway
dhcp-option=3,192.168.1.1

#Sets DNS server 0.0.0.0,8.8.8.8 would query the local machine and the the next address
dhcp-option=6,192.168.1.1

#Sets additional name server, probably not needed, need to test this
server=8.8.8.8

#Logs, always good to have logs
log-queries
log-dhcp
>> Assign the gateway and netmask to the interface 

```

ifconfig [interface] up 192.168.1.1 netmask 255.255.255.0

```

>> Add to routing table

```
route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.1.1

```
##Error Troubleshooting##
##iptables/1.8.2 Failed to initialize nft: Protocol not supported##

```

update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
update-alternatives --set arptables /usr/sbin/arptables-legacy
update-alternatives --set ebtables /usr/sbin/ebtables-legacy

```

##################################################################################################
Most likely though you just need to restart after running all updates and upgrades, as the kernel
was probably updated.
###################################################################################################
>> Start dnsmasq server protocol

#-C flag specifies the conf file and -d is debug mode, stops dnsmasq from forking into the background

```
dnsmasq -C /etc/dnsmasq.conf -d

```
Route traffic:

>> Traffic forwarding to give internet access to the clients that are connecting

#
```

iptables --table nat --append POSTROUTING --out-interface [egress interface] -j MASQUERADE

```

#
```

iptables --append FORWARD --in-interface [interface that is in monitor] -j ACCEPT
Enable IP forwarding

```

#Enables IP forwarding

```
echo 1 > /proc/sys/net/ipv4/ip_forward

```

##Alternatively##

```
sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward=1

```
