```
ip address print
ip route print
ip dhcp-client print 
ip address add address=192.168.0.1 netmask=255.255.255.0 interface=ether1
ip route add dst-address=192.168.0.0/24 gateway=ether1
ip pool add name=___ range=...-...
ip dhcp-server network add address=10.0.3.0/24 dns-server=8.8.8.8,4.4.4.4 gateway=10.0.3.254
ip dhcp-server add interface=ether1 address-pool=pool lease-time=24h name=dhcp disabled=no

routing rip interface add interface=ether2 send=v2 receive=v2

ip firewall nat add chain=srcnat action=masquerade out-interface=ether3
```

на компах GNS3 `show ip`

