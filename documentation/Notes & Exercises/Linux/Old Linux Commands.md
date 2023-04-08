## Ubuntu

Modifying network settings

```bash
vi /etc/network/interfaces

#
iface eth0 inet static
address 192.168.1.5
netmask 255.255.255.0
gateway 192.168.1.254
#

sudo /etc/init.d/networking restart
```

