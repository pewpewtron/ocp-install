# Prerequisite before deploying OCP cluster

SVC/Helper node must connect to 2 network one is use for internal ocp cluster and the other is used for external connection.

## Networking setups

### 1. Install and configure Firewalld

Create 2 connection zone for internal and external network.

```bash
ifconfig
nmcli connection modify <network-interface> connection.zone internal
nmcli connection modify <network-interface> connection.zone external
```

Set masquerading (source-nat) on the both zones.

```bash
firewall-cmd --zone=internal --add-masquerade --permanent
firewall-cmd --zone=external --add-masquerade --permanent

# reload settings
firewall-cmd --reload

# check settings
firewall-cmd --list-all --zone=internal
firewall-cmd --list-all --zone=external

# check ipforwarding
cat /proc/sys/net/ipv4/ip_forward
```

### 2. Install and configure Bind DNS

```bash
dnf install bind bind-utils -y

# Apply config from template
cp network-config/dns/named.conf /etc/named.conf
cp -R network-config/dns/zones /etc/named/

# Configure the firewall for DNS
firewall-cmd --add-port=53/udp --zone=internal --permanent
firewall-cmd --add-port=53/tcp --zone=internal --permanent
firewall-cmd --reload

# Enable and start service
systemctl enable named
systemctl start named
systemctl status named
```

> At the moment DNS will still be pointing to the LAN DNS server. You can see this by testing with `dig ocp-svc.lab.i3datacenter.my.id`.
> Change the LAN nic (ens192/used for external network) to use 127.0.0.1 for DNS AND ensure `Ignore automatically Obtained DNS parameters` is ticked using `nmtui`

Restart NetworkManager and check dig sees the correct DNS results by using the DNS Server running locally then check using dig command

```bash
systemctl restart NetworkManager
dig ocp-svc.lab.i3datacenter.my.id
dig -x 192.168.22.40
```

### 3. Install and configure DHCP

```bash
dnf install dhcp-server -y
```

Edit dhcp config file located on `/etc/dhcp/dhcpd.conf`
after dhcp config is modified configure the firewall in internal zone
then enable and start the service

```bash
firewall-cmd --add-service=dhcp --zone=internal --permanent
firewall-cmd --reload

# Starting DHCP service
systemctl enable dhcpd
systemctl start dhcpd
systemctl status dhcpd
```

### 4. Install and configure HAProxy

```bash
dnf install haproxy -y
```

Edit HAProxy config file located on `/etc/haproxy/haproxy.cfg`
after editing HAProxy config file configure the firewall

>Note: Opening port 9000 in the external zone allows access to HAProxy stats that are useful for monitoring and troubleshooting. The UI can be accessed at: `http://{ocp-svc_IP_address}:9000/stats`

```bash
# kube-api-server on control plane nodes
firewall-cmd --add-port=6443/tcp --zone=internal --permanent 
# kube-api-server on control plane nodes
firewall-cmd --add-port=6443/tcp --zone=external --permanent 
# machine-config server
firewall-cmd --add-port=22623/tcp --zone=internal --permanent
# web services hosted on worker nodes 
firewall-cmd --add-service=http --zone=internal --permanent 
# web services hosted on worker nodes
firewall-cmd --add-service=http --zone=external --permanent 
# web services hosted on worker nodes
firewall-cmd --add-service=https --zone=internal --permanent 
# web services hosted on worker nodes
firewall-cmd --add-service=https --zone=external --permanent
# HAProxy Stats 
firewall-cmd --add-port=9000/tcp --zone=external --permanent 
firewall-cmd --reload
```

Enable and start the service

```bash
# SELinux name_bind access
setsebool -P haproxy_connect_any 1

systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
```

## Service setups

### 1. Install & configure Apache Web Server

```bash
dnf install httpd -y
```

Change default listen port to 8080 in httpd.conf located on `/etc/httpd/conf/httpd.conf`

```bash
vim /etc/httpd/conf/httpd.conf
# Or
sed -i 's/Listen 80/Listen 0.0.0.0:8080/' /etc/httpd/conf/httpd.conf
```

Configure the firewall for Web Server traffic

```bash
firewall-cmd --add-port=8080/tcp --zone=internal --permanent
firewall-cmd --reload
```

Enable and start the service

```bash
systemctl enable httpd
systemctl start httpd
systemctl status httpd
```

### 2. Install & configure NFS Server

```bash
dnf install nfs-utils -y
```

Create the Share
Check available disk space and its location `df -h`

```bash
mkdir -p /shares/registry
chown -R nobody:nobody /shares/registry
chmod -R 777 /shares/registry
```

Export the Share

```bash
echo "/shares/registry  192.168.22.0/24(rw,sync,root_squash,no_subtree_check,no_wdelay)" > /etc/exports
exportfs -rv
```

Set Firewall rules:

```bash
firewall-cmd --zone=internal --add-service mountd --permanent
firewall-cmd --zone=internal --add-service rpc-bind --permanent
firewall-cmd --zone=internal --add-service nfs --permanent
firewall-cmd --reload
```

Enable and start the NFS related services

```bash
systemctl enable nfs-server rpcbind
systemctl start nfs-server rpcbind nfs-mountd
```
