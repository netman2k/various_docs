# Pulp install on CentOS7

[Installation](http://docs.pulpproject.org/user-guide/installation/f23-.html)

## Set Hostname
```
hostnamectl set-hostname pulp.<Your domain>
```

## Set SELinux
```
sed -i s/SELINUX=enforcing/SELINUX=permissive/g /etc/selinux/config
setenforce 0
```

## Install Pulp repository
```
cat <<'EOF' > /etc/yum.repos.d/pulp.repo
[pulp-2-stable]
name=Pulp 2 Production Releases
baseurl=https://repos.fedorapeople.org/repos/pulp/pulp/stable/2/$releasever/$basearch/
enabled=1
skip_if_unavailable=1
gpgcheck=1
gpgkey=https://repos.fedorapeople.org/repos/pulp/pulp/GPG-RPM-KEY-pulp-2
EOF
```

## Install EPEL
```
yum install http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm -y
```


## Install MongoDB
```
yum install mongodb-server -y
systemctl start mongod && systemctl enable $_
```

> **TODO** SSL & authorization configuration|


## Install QPID
```
yum install qpid-cpp-server qpid-cpp-server-linearstore -y
systemctl start qpidd && systemctl enable $_
```

## Install Pulp server, task workers and dependencies
```
yum install pulp-server python-gofer-qpid python-qpid qpid-tools -y
```

## Install support for different content via plugins
```
yum install pulp-rpm-plugins pulp-puppet-plugins pulp-docker-plugins -y
```

Install admin client
```
yum install pulp-admin-client pulp-rpm-admin-extensions \
pulp-puppet-admin-extensions pulp-docker-admin-extensions -y
```

## Edit /etc/pulp/server.conf 

if you set SSL on QPID, Database and etc. edit the server.conf file.

TODO

## Generate RSA key pair and SSL CA certificate
```
pulp-gen-key-pair
pulp-gen-ca-certificate
```

## Initialize Pulp's database
```
sudo -u apache pulp-manage-db
```

## Disable SSLv3
```
sed -i 's/^\(SSLProtocol all -SSLv2\)$/\1 -SSLv3/' /etc/httpd/conf.d/ssl.conf
```

## Start Apache
```
systemctl start httpd && systemctl enable $_
```

## Start Celery
```
systemctl start pulp_workers && systemctl enable $_
systemctl start pulp_celerybeat && systemctl enable $_
systemctl start pulp_resource_manager && systemctl enable $_
```

## Set pulp client setting
```
mkdir ~/.pulp
cat <<EOF > ~/.pulp/admin.conf 
[server]
verify_ssl = False
host = $(hostname -f)

[auth]
username: admin
password: admin
EOF

chmod 600 ~/.pulp/admin.conf
```
[Repositoies](https://docs.pulpproject.org/user-guide/admin-client/repositories.html)
## Create & Sync Repostories 
[http://docs.pulpproject.org/plugins/pulp_rpm/user-guide/quick-start.html]

```
pulp-admin rpm repo create --repo-id=centos7_os --relative-url=centos/7/os/x86_64 --feed=http://mirror.oasis.onnetcorp.com/centos/7/os/x86_64/
pulp-admin rpm repo create --repo-id=centos7_updates --relative-url=centos/7/updates --feed=http://mirror.oasis.onnetcorp.com/centos/7/updates/x86_64/
pulp-admin rpm repo create --repo-id=centos7_extras --relative-url=centos/7/extras --feed=http://mirror.oasis.onnetcorp.com/centos/7/extras/x86_64/
pulp-admin rpm repo create --repo-id=puppet5_el7 --relative-url=puppet5/el/7/x86_64 --feed=http://yum.puppetlabs.com/puppet5/el/7/x86_64
pulp-admin rpm repo sync run --repo-id=centos7_os
pulp-admin rpm repo sync run --repo-id=centos7_updates
pulp-admin rpm repo sync run --repo-id=centos7_extras
pulp-admin rpm repo sync run --repo-id=puppet5_el7
```

## Access URL
> https://10.40.205.236/pulp/repos/centos/7/

## Sync Scheduling

[recipes](http://docs.pulpproject.org/plugins/pulp_rpm/user-guide/recipes.html)

### sync per 1w starts from 2017-08-11
```
pulp-admin rpm repo sync schedules create -s '2017-08-11T00:00Z/P1W' --repo-id=centos7_os 
```
### sync updates repo per 1w starts from 2017-08-12
```
pulp-admin rpm repo sync schedules create -s '2017-08-12T00:00Z/P1W' --repo-id=centos7_updates 
```
### sync extras repo per 1w starts from 2017-08-13
```
pulp-admin rpm repo sync schedules create -s '2017-08-13T00:00Z/P1W' --repo-id=centos7_extras 
```
### sync puppet5 repo per 1w starts from 2017-08-11
```
pulp-admin rpm repo sync schedules create -s '2017-08-11T00:00Z/P1W' --repo-id=puppet5_el7
```

