> HIGLY EXPERIMENTAL TASK - NOT WORK

---
이 페이지는 다음과 같은 사항을 기록하기 위한 용도록 만들어짐

외부 데이터베이스 사용 테스트
다중 마스터 구현 가능 여부 테스트
플러그인 설정
---

# Installing Katello 3.3 with an external PostgreSQL

## On PostgreSQL server
 
TODO

## On Katello Server
### OS Settings
#### hostname
> Use service oriented hostname for HA
> In this guide, I used a virtual hostname, cause I will use a load balancer on top of the katello servers.
> If you want to use multiple Katello servers on your environment, you need to use a single service domain name (eg, katello.example.local)
> after installing, you could set your origin hostname as normal (eg, katello01.example.local)

Before running installer, you need to set the hostname properly.
```
[root@localhost ~]# hostnamectl set-hostname katello01.example.local
[root@localhost ~]# systemctl restart systemd-hostnamed.service
```
You could get the IP by anyway:
```
IP=$(ip addr |grep eth0: -A 2 | tail -n 1 | awk '{ gsub("/24","",$2); print $2 }')
echo "${IP} $(hostname -f) katello"  >> /etc/hosts
```
#### NTP
> You could use ntpd instead of chrony.
```
yum install chrony -y
systemctl enable chronyd.service;systemctl start chronyd.service
chronyc tracking
```

#### Firewall
[Go to the firewall settings](firewall.md)

### Installing Katello
Register RPM Repositories
[Go to the repository settings](repository.md)

#### Installing installer
The katello RPM will install all requirements
```
yum -y install katello
```

Fixing dependency problem with Puppet 4 (or PuppetServer 2.x)
[Go to the repository settings](fix_puppet4_issue.md)


#### Editing answer file
Before editing an answer file, you would better back the file up first.
```
cp /etc/foreman-installer/scenarios.d/katello-answers.yaml{,.back}
```

#### Setting for self-signed CA
The foreman-installer create a CA with pre-defined settings which will not suit your environment.

Let's change this preset by adding these lines
```
certs:
  country: KR
  state: Seoul
  city: Seoul
  org: EXAMPLE LOCAL
  org_unit: OPS team
```

#### Database setting
The workload on a database is quite higher than the others. I recommend that you use external database instead of local one.

Katello needs two databases which are for the candlepin and foreman services.

> I assumed that you've already created these databases on external postgresql database server.

/etc/foreman-installer/scenarios.d/katello-answer.yaml
```
foreman:
  db_manage: true
  db_type: postgresql
  db_adapter:
  db_host: "172.16.0.187"
  db_port:
  db_database: db_katello
  db_username: katello
  db_password: please_change_me
  db_sslmode:
  db_pool: 5
  admin_password: please_change_me
  admin_email: ops@example.local
```

#### Set Candlepin Database of Candlepin
> Wrote date: 2017.02.13
So far, there is no way to make candlepin to use an external database neither answer or hiera with the foreman-installer command.
To be able to use the external database I changed the source code as below:

postgresql.pp
```
daehyung@daehyung-desktop:~/git/github.com/Foreman/puppet-candlepin$ git diff b9fdb665ed780ea8ce27a7670ca5200bbd1440ad 6d0c1269cf8c3928b8bc7f2e667ffdb2304dedcd
diff --git a/manifests/database/postgresql.pp b/manifests/database/postgresql.pp
index 360bfa4..2acdd5e 100644
--- a/manifests/database/postgresql.pp
+++ b/manifests/database/postgresql.pp
@@ -26,33 +26,34 @@ class candlepin::database::postgresql{
     # Temporary direct use of liquibase to initiall migrate the candlepin database
     # until support is added in cpdb - https://bugzilla.redhat.com/show_bug.cgi?id=1044574
     include ::postgresql::client, ::postgresql::server
-    postgresql::server::db { $candlepin::db_name:
-      user     => $candlepin::db_user,
-      password => postgresql_password($candlepin::db_user, $candlepin::db_password),
+    postgresql::server::db { $db_name:
+      user     => $db_user,
+      password => postgresql_password($db_user, $db_password),
       encoding => 'utf8',
       locale   => 'en_US.utf8',
-    } ~>
-    exec { 'cpdb':
-      path        => '/bin:/usr/bin',
-      command     => "liquibase --driver=org.postgresql.Driver \
-                            --classpath=/usr/share/java/postgresql-jdbc.jar:/var/lib/${candlepin::tomcat}/webapps/candlepin/WEB-INF/classes/ \
-                            --changeLogFile=db/changelog/changelog-create.xml \
-                            --url=jdbc:postgresql:${candlepin::db_name} \
-                            --username=${candlepin::db_user}  \
-                            --password=${candlepin::db_password} \
-                            migrate \
-                            -Dcommunity=False \
-                            >> ${candlepin::log_dir}/cpdb.log \
-                            2>&1 && touch /var/lib/candlepin/cpdb_done",
-      creates     => "${candlepin::log_dir}/cpdb_done",
-      refreshonly => true,
-      before      => Service[$candlepin::tomcat],
-      require     => [
-        Package['candlepin'],
-        Concat['/etc/candlepin/candlepin.conf']
-      ],
+      before   => Exec['cpdb'],
     }
-
-    Postgresql::Server::Role[$candlepin::db_user] -> Postgresql::Server::Database[$candlepin::db_name]
+    Postgresql::Server::Role[$db_user] -> Postgresql::Server::Database[$db_name]
+  }
+  
+  exec { 'cpdb':
+    path        => '/bin:/usr/bin',
+    command     => "liquibase --driver=org.postgresql.Driver \
+                          --classpath=/usr/share/java/postgresql-jdbc.jar:/var/lib/${candlepin::tomcat}/webapps/candlepin/WEB-INF/classes/ \
+                          --changeLogFile=db/changelog/changelog-create.xml \
+                          --url=jdbc:postgresql://${db_host}:${db_port}/${db_name} \
+                          --username=${db_user}  \
+                          --password=${db_password} \
+                          migrate \
+                          -Dcommunity=False \
+                          >> ${candlepin::log_dir}/cpdb.log \
+                          2>&1 && touch /var/lib/candlepin/cpdb_done",
+    creates     => "${candlepin::log_dir}/cpdb_done",
+    refreshonly => true,
+    before      => Service[$candlepin::tomcat],
+    require     => [
+      Package['candlepin'],
+      Concat['/etc/candlepin/candlepin.conf']
+    ],
   }
 }
```
After changing or copying the file /usr/share/katello-installer-base/modules/candlepin/manifests/database/postgresql.pp
Now you should set the hiera data as below:
```
cp /usr/share/foreman-installer/config/foreman.hiera/custom.yaml{,.back}
cat <<'EOF' >> /usr/share/foreman-installer/config/foreman.hiera/custom.yaml
candlepin::manage_db: false
candlepin::db_type: postgresql
candlepin::db_host: 172.16.0.187
candlepin::db_name: db_candlepin
candlepin::db_user: candlepin
candlepin::db_password: please_change_me
EOF
```

> You should apply this on your PostgreSQL server.

#### Set smart proxy bind option
I get this error "TCPServer Error: Address already in use - bind(2)" on the cloudstack environment which does not support IPv6 at this moment.

So I added this option to use IPv4 only.
```
foreman_proxy:
  # Fix IPv6 bind error
  bind_host: 0.0.0.0
```

#### Set Puppet related options
Normally I did set puppet related settings with foreman-installer command, but it is not useful when you need to install katello multiple times.

so I added these kind options in an answer file as below.

#### Set Puppet common name
I will use a common name for my puppet servers. I do not want to remember or use every puppet servers' name.

this setting will set subject alternative names on the certificate of the puppet server.

/etc/foreman-installer/scenarios.d/katello-answer.yaml
```
# This value will prevent the puppet redeclaration problem
foreman_proxy_content:
 puppet: false
puppet:
 dns_alt_names:
  - <Smart Proxy's FQDN>
  - puppetca-lb.example.local
  - puppetca.example.local
  - puppet-lb.example.local
  - puppet.example.local
  - puppet
```
---
> Note that I set on the puppet option of the foreman_proxy_content to false .

cause the foreman-install with katello scenario calls the puppet class automatically when the puppet feature is active on the katello server
```
  if $puppet {
    class { '::certs::puppet':
      hostname => $foreman_proxy_fqdn,
    } ~>
    class { '::puppet':
      server                      => true,
      server_ca                   => $::foreman_proxy::puppetca,
      server_foreman_url          => $foreman_url,
      server_foreman_ssl_cert     => $::certs::puppet::client_cert,
      server_foreman_ssl_key      => $::certs::puppet::client_key,
      server_foreman_ssl_ca       => $::certs::puppet::ssl_ca_cert,
      server_storeconfigs_backend => false,
      server_dynamic_environments => true,
      server_environments_owner   => 'apache',
      server_config_version       => '',
      server_enc_api              => 'v2',
      server_ca_proxy             => $puppet_ca_proxy,
      server_implementation       => $puppet_server_implementation,
      additional_settings         => {
                                        'disable_warnings' => 'deprecations',
      },
    }
  }
```
And the above code will make an error like this
```
[ERROR 2017-02-10 02:13:25 main]  Evaluation Error: Error while evaluating a Resource Statement, Duplicate declaration: Class[Puppet] is already declared; cannot redeclare at /usr/share/katello-installer-base/modules/foreman_proxy_content/manifests/init.pp:225 at /usr/share/katello-installer-base/modules/foreman_proxy_content/manifests/init.pp:225:5 on node katello.example.local
```
> I found that this setting does not cause any functional problems. but you should remember!
> It can be changed any time as time goes by...

After installing, you would check the subject alternative name by running this command
```
[root@katello ~]# openssl x509 -noout -text -in /etc/puppetlabs/puppet/ssl/certs/$(hostname -f).pem  |grep "X509v3 Subject Alternative Name" -A 1
            X509v3 Subject Alternative Name: 
                DNS:katello.example.local, DNS:katello01.example.local, DNS:puppet, DNS:puppet.example.local, DNS:puppetca.example.local
```

#### Set Puppet directory settings
Since Puppet 4, the home directory of the puppet is set to /etc/puppetlabs/puppet.
[https://docs.puppet.com/puppet/4.8/dirs_confdir.html]
```
foreman:
  puppet_home: /etc/puppetlabs/puppet
```

#### Set Puppet SSL certificate settings
*No longer set these value manually, the foreman-installer detect it automatically.*

#### Set PuppetCA related settings
*No longer set these value manually, the foreman-installer detect it automatically.*

#### Set PuppetDB related settings

[Integrating with PuppetDB](puppetdb_integration.md)

### Run Installer
The installation may be customized , to see a list of options foreman-installer --scenario katello --help.

At this time I'll choose the default one:
```
foreman-installer --scenario katello
```

### Self register
There are several advantages to registering it as a client to itself:

* The same lifecycle management procedures can be applied to the Katello 6 server itself that have been applied to the rest of the managed estate.
* By subscribing the Katello 6 server to its own content views, it will receive the same updates on the same schedule as the rest of the managed hosts.
* You can use virt-who by using a self-registered Katello Server, but you can also install and configure virt-who without self-registration.

But there are also several limitations of a self-registered Katello server:

* A self-registered Katello server cannot test package updates by using life-cycle environments. It is essential to make a full backup of a self-registered Katello server before doing an upgrade to untested packages.
* Not all puppet modules are supported by a self-registered Katello server. When applying puppet modules to a self-registered Katello server ensure that they will not create an unsupported configuration.
```
yum install /var/www/html/pub/katello-ca-consumer-latest.noarch.rpm
subscription-manager register --org="Default_Organization" --environment=Library
subscription-manager list --available --all
subscription-manager attach --pool <Pool ID>
yum install katello-agent -y
```
Example
```
[root@katello01 puppetdb_foreman]# rpm -Uvh /var/www/html/pub/katello-ca-consumer-latest.noarch.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:katello-ca-consumer-katello01.cdn################################# [100%]

[root@katello01 puppetdb_foreman]# subscription-manager register --org="Default_Organization" --environment=Library
Registering to: katello01.example.local:443/rhsm
Username: admin
Password: 
The system has been registered with ID: 96c133a9-593d-4149-ad5d-1a55f1d6dcee 


[root@katello01 puppetdb_foreman]# subscription-manager list --available --all
+-------------------------------------------+
    Available Subscriptions
+-------------------------------------------+
Subscription Name:   katello-client-3.2-el7
Provides:            
SKU:                 1481850582496
Contract:            
Pool ID:             2c9080d75901ee4b0159052d77580046
Provides Management: No
Available:           Unlimited
Suggested:           1
Service Level:       
Service Type:        
Subscription Type:   Standard
Ends:                2046년 12월 09일
System Type:         Physical
Subscription Name:   PostgreSQL 9.6
Provides:            
SKU:                 1481786502154
Contract:            
Pool ID:             2c9080d759014d660159015bad900003
Provides Management: No
Available:           Unlimited
Suggested:           1
Service Level:       
Service Type:        
Subscription Type:   Standard
Ends:                2046년 12월 08일
System Type:         Physical
Subscription Name:   Puppet Labs PC1
Provides:            
SKU:                 1480981700035
Contract:            
Pool ID:             2c9080d758ce64540158d1635f920002
Provides Management: No
Available:           Unlimited
Suggested:           1
Service Level:       
Service Type:        
Subscription Type:   Standard
Ends:                2046년 11월 28일
System Type:         Physical
Subscription Name:   CentOS 7
Provides:            
SKU:                 1480390485479
Contract:            
Pool ID:             2c9080d758ae21160158ae2627c40007
Provides Management: No
Available:           Unlimited
Suggested:           1
Service Level:       
Service Type:        
Subscription Type:   Standard
Ends:                2046년 11월 22일
System Type:         Physical
Subscription Name:   EPEL-7
Provides:            
SKU:                 1481786615271
Contract:            
Pool ID:             2c9080d759014d660159015d66c6000a
Provides Management: No
Available:           Unlimited
Suggested:           1
Service Level:       
Service Type:        
Subscription Type:   Standard
Ends:                2046년 12월 08일
System Type:         Physical

[root@katello01 puppetdb_foreman]# subscription-manager attach --pool 2c9080d75901ee4b0159052d77580046 --pool 2c9080d758ce64540158d1635f920002 --pool 2c9080d758ae21160158ae2627c40007 --pool 2c9080d759014d660159015d66c6000a
Successfully attached a subscription for: katello-client-3.2-el7
Successfully attached a subscription for: Puppet Labs PC1
Successfully attached a subscription for: CentOS 7
Successfully attached a subscription for: EPEL-7
[root@katello01 puppetdb_foreman]# yum install katello-agent -y
<SNIP>
```

### Enable foreman plugins repo.
[https://www.theforeman.org/plugins/#2.2Packageinstallation]

To enable foreman plugins repo., you should create a repo. file as below:
```
cat <<'EOF' > /etc/yum.repos.d/foreman-plugins.repo
[foreman-plugins]
name=Foreman plugins
baseurl=http://yum.theforeman.org/plugins/1.13/el7/x86_64/
enabled=1
gpgcheck=0
EOF
```
