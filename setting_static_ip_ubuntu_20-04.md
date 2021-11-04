# Setting Static IP Ubuntu Server 20.04

## Before we begin

### What's our network interface name?

Enter the foloowing command to get information including the interface name of the network interface we intend on setting up with a staic ip.  
  
```shell
ip a
```
On my system it is eno1 so thats what i will be using in this example however this will most likely **NOT** match your system.

#### If server was setup using cloudinit it needs to be disabled
To disable cloudinit we have to create the following file:
```shell
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```
```yaml
/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
network: {config: disabled}
```
### Setting a static IP using Netplan

Create a new file in the /etc/netplan folder called `01-netcfg.yaml'
```shell
sudo nano /etc/netplan/01-netcfg.yaml
```
Copy the following code into the new file
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: yes
```

> If you intend on using Cockpit to montior and manage package updates you will want to change `networkd` to `NetworkManager`.
  


At this point if you were to apply `netplan` you would end up with a DHCP address. This is a default configuration. We will now modify the file to match the following code and to specify our static IP config.
  


```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: no
      addresses:
        - 192.168.1.10/24
      gateway4: 192.168.1.1
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
```
  

> Again change *eno1* to match YOUR network interface name found with the `ip a` command. Also notice that im using both Googles (8.8.8.8) and Cloudflare (1.1.1.1) DNS servers here, feel free to change to whatever you prefer.
>
>>Take note of the changed '*dhcp4:*' value. The IP address must be entered in [CIDR](https://account.arin.net/public/cidrCalculator) notation.
  
Once done, save the file and apply the changes by running the following command:
  
```bash
sudo netplan apply
```

To verify that our new IP address is set enter the following command

```shell
ip a show dev eno1
```