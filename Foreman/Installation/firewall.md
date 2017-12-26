# Firewall Settings

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html]
[https://access.redhat.com/documentation/en/red-hat-satellite/6.2/paged/installation-guide/chapter-2-preparing-your-environment-for-installation]

> Note that it's not fully tested!!

The following ports need to be open to external connections:

* 80 TCP - HTTP, used for provisioning purposes
* 443 TCP - HTTPS, used for web access and api communication
* 5647 TCP - qdrouterd - used for client and capsule actions
* 9090 TCP - HTTPS - used for communication with the smart proxy

**Check current available services and enabled services**
I set the public as default zone, you need to change as if you installed katello server in the different network such as DMZ or internal network.
```
firewall-cmd --get-services
amanda-client bacula bacula-client dhcp dhcpv6 dhcpv6-client dns ftp high-availability http https imaps ipp ipp-client ipsec kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxy pmwebapi pmwebapis pop3s postgresql proxy-dhcp radius rpc-bind samba samba-client smtp squid ssh telnet tftp tftp-client transmission-client vnc-server wbem-https
firewall-cmd --zone=public --list-services
dhcpv6-client ssh
```
**Enable predefined services (80,443)**
```
firewall-cmd --zone=public --add-service http
firewall-cmd --zone=public --add-service https
firewall-cmd --zone=public --permanent
```
**Adding other services (5647, 9090)**
I create other services to allow traffics
```
cat <<EOF > /etc/firewalld/services/capsule.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Katello - Capsule</short>
  <description>qdrouterd - used for client and capsule actions</description>
  <port protocol="tcp" port="5647"/>
</service>
EOF

cat <<EOF > /etc/firewalld/services/smartproxy.xml 
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Katello - SmartProxy</short>
  <description>HTTPS - used for communication with the smart proxy</description>
  <port protocol="tcp" port="9090"/>
</service>
EOF

firewall-cmd --reload
```

**Firewall reloading & enabling the other services**
As you can see, the added services are visible such as capsule and smartproxy.

```
firewall-cmd --zone=public --add-service capsule
firewall-cmd --zone=public --add-service smartproxy 
firewall-cmd --zone=permanent

firewall-cmd --zone=public --list-services
capsule dhcpv6-client http https smartproxy ssh

firewall-cmd --zone=public --permanent
```


