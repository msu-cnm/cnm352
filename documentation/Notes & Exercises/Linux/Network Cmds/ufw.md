## Uncomplicated Firewall

Disable IPv6 for UFW

```bash
vi /etc/default/ufw
IPV6=no
sudo ufw reload
```

Enable/Disable UFW

```bash
sudo ufw enable

sudo ufw disable
```

Allow a port (both IPv4 and IPv6)

```bash
sudo ufw allow PORT
```

Allow a port for a specific inbound IP

```bash
sudo ufw allow from 10.11.1.223/32 to any port 9999
```

Allow a port list

```bash
sudo ufw allow proto udp to any port 1194,4569,5060
```

Allow on specific interface

```bash
sudo ufw allow out on enp0s17
```

Delete a rule

```bash
# Enter the rule number to delete or spell out the rule
sudo ufw delete 1
```



Example (Lubuntu Rules)

```bash
sudo ufw allow out on enp0s17 from any to 10.0.2.2
sudo ufw allow out on enp0s17 proto udp to any port 1194,4569,5060
sudo ufw deny out on enp0s17
sudo ufw reload
```

