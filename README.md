# NoteNetwork Ecorouter
####### HQ #######

en
conf t

hostname hq-rtr
ip domain-name au.team

username sshuser
password P@$$w0rd
role admin
exit
admin

do show port brif
do show port ge0
do show port ge1

int ISP
ip add 4.4.4.2/28
no sh
exit

port ge0
service-instance Internet
encapsulation untagged
connect ip interface ISP
exit
exit

ip route 0.0.0.0/0 4.4.4.1

int v10
ip address 192.168.101.30/27
no sh
exit

int v20
ip address 192.168.100.254/24
no sh
exit

int v99
ip address 192.168.101.38/29
no sh
exit

port ge1
service-instance ge1/vlan10
encapsulation dot1q 10 exact
rewrite pop 1
connect ip interface v10
exit
exit

port ge1
service-instance ge1/vlan20
encapsulation dot1q 20 exact
rewrite pop 1
connect ip interface v20
exit
exit

port ge1
service-instance ge1/vlan99
encapsulation dot1q 99 exact
rewrite pop 1
connect ip interface v99
exit
exit

do sh ip int br
do sh port br

int ISP
ip nat outside
exit

int v10
ip nat inside
exit

int v20
ip nat inside
exit

int v99
ip nat inside
exit


ip nat pool NAT 192.168.100.1-192.168.100.254,192.168.101.1-192.168.101.30,192.168.101.33-192.168.101.38

ip nat source dynamic inside pool NAT overload 4.4.4.2

dhcp-profile 0
server 192.168.101.1
mode relay
exit

interface v20
dhcp-profile 0
exit

interface v99
dhcp-profile 0
exit



####### DC #######

en
conf t

hostname dc-rtr
ip domain-name au.team

username sshuser
password P@$$w0rd
role admin
exit

do show port brif
do show port ge0
do show port ge1

int ISP
ip add 6.6.6.2/29
no sh
exit

port ge0
service-instance Internet
encapsulation untagged
connect ip interface ISP
exit
exit

ip route 0.0.0.0/0 6.6.6.1

int LAN
ip add 172.30.20.1/22
no sh
exit

port ge1
service-instance Internet
encapsulation untagged
connect ip interface LAN
exit
exit

int ISP
ip nat outside
exit

int LAN
ip nat inside
exit

ip nat pool NAT 172.30.20.1-172.30.23.254

ip nat source dynamic inside pool NAT overload 6.6.6.2


###### HQ ######

interface tunnel.1
ip add 10.10.10.1/30
ip mtu 1400
ip tunnel 4.4.4.2 6.6.6.2 mode gre
exit


###### DC ######

interface tunnel.1
ip add 10.10.10.2/30
ip mtu 1400
ip tunnel 6.6.6.2 4.4.4.2 mode gre
exit


###### HQ ######

router ospf 1
network 10.10.10.0/30 area 0
network 192.168.100.0/24 area 0
network 192.168.101.0/27 area 0
network 192.168.101.33/29 area 0
passive-interface default
no passive-interface tunnel.1
exit


###### DC ######

router ospf 1
network 10.10.10.0/30 area 0
network 172.30.20.0/22 area 0
passive-interface default
no passive-interface tunnel.1
exit

###### DC ######

ip nat source static tcp 172.30.20.2 3000 4.4.4.2 3000 

