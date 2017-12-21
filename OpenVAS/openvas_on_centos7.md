# OpenVAS Installation
[OpenVAS](http://www.openvas.org)

The world's most advanced Open Source vulnerability scanner and manager

OpenVAS is a framework of several services and tools offering a comprehensive and powerful vulnerability scanning and vulnerability management solution.

## Installing
### Set SELinux

OpenVAS requires SELinux to be disabled.

```
setenforce 0
sed -i '/SELINUX=/s/enforcing/disabled/' /etc/selinux/config
```

### Set Repository

```
wget -q -O - http://www.atomicorp.com/installers/atomic | sh
```

### Install packages
```
yum install wget bzip2 net-tools alien gnutls-utils -y
yum install texlive-latex-bin-bin texlive-comment texlive-titlesec texlive-changepage -y 
yum install openvas -y
```

Texlive packages will be used to generate PDF report without it you can not generate PDF fucntion.

### Downgrade NMAP

OpenVAS requires NMAP 5.51, so we should install nmap manually and lock this version from updating.
```
yum install https://nmap.org/dist/nmap-5.51.6-1.x86_64.rpm
yum install yum-plugin-versionlock
yum versionlock 2:nmap-5.51.6-1.x86_64
```

### Set Redis

Edit /etc/redis.conf file and uncommenting these lines
```
    unixsocket /tmp/redis.sock
    unixsocketperm 700
```
After editing, restart redis service
```
systemctl enable redis && systemctl restart redis
```

### Set Firewall
```
firewall-cmd --permanent --zone=public --add-port=9392/tcp
firewall-cmd --reload
```

## Configuring OpenVAS
#### (Optional) Initial Feeds

This processes will do later when you run openvas-setup but we would better check it first.
```
greenbone-nvt-sync
greenbone-certdata-sync
greenbone-scapdata-sync
```

Make sure your system can communicate with the feed.openvas.org site via 873/tcp port that is used by rsync.

### Rebuild manager database
```
openvasmd --rebuild
```

### Setup
```
openvas-setup
```

You should note your admin account's info.

Now you can connect to OpenVAS Web UI via https://<Your IP>:9392

## Verify installation

```
openvas-check-setup --v9
    WARNING: Signature checking of NVTs is not enabled in OpenVAS Scanner.
    SUGGEST: Enable signature checking (see http://www.openvas.org/trusted-nvts.html).
```
     