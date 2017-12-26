## Enabling Puppet report to PuppetDB
### Method 1 (Recommended)
*This method is recommanded when you install foreman first time.*

#### Editing the answer file
Open the katello-answers file from the directory /etc/foreman-installer/scenarios.d and add these:
```
puppet:
  server_reports: foreman,puppetdb
  server_puppetdb_host: <YOUR PUPPETDB FQDN>
puppet::server:
  storeconfigs_backend: puppetdb
puppetdb::master::config::strict_validation: false
```
the rest parts are the same

### Method 2
This method is the way when you already installed Foreman or Katello server.

Set Hiera
If you are willing to integrate Foreman/Katello with PuppetDB, you should set these value in the hiera file (/usr/share/foreman-installer/config/foreman.hiera/RedHat.yaml).
I found that there's no way to set PuppetDB related settings via foreman-installer.
```
cat <<EOF >> /usr/share/foreman-installer/config/foreman.hiera/RedHat.yaml
puppet::server_reports: puppetdb,foreman
puppet::server_puppetdb_host: puppetdb.$(facter domain)
puppet::server::storeconfigs_backend: puppetdb
puppetdb::master::config::strict_validation: false
EOF
```
> Note that
> I just want to add PuppetDB specific configuration into the puppet server's configuration.
> so I set puppetdb::master::config::strict_validation to false. it makes puppetdb module to skip connection validation between Foreman/Katello(actually PuppetServer)  and PuppetDB.

#### Updating Puppet configuration
foreman-installer --scenario katello --upgrade-puppet

---
You could see this kind of message. but it's OK, it happens due to the existence of the object.
```
 qpid-config --ssl-certificate /etc/pki/katello/certs/java-client.crt --ssl-key /etc/pki/katello/private/java-client.key -b 'amqps://katello.example.local:5671' add exchange topic event --durable returned 1 instead of one of [0]
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/util/errors.rb:106:in `fail'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/util/errors.rb:106:in `fail'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/type/exec.rb:160:in `sync'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction/resource_harness.rb:236:in `sync'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction/resource_harness.rb:134:in `sync_if_needed'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction/resource_harness.rb:88:in `block in perform_changes'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction/resource_harness.rb:87:in `each'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction/resource_harness.rb:87:in `perform_changes'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction/resource_harness.rb:21:in `evaluate'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction.rb:230:in `apply'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction.rb:246:in `eval_resource'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction.rb:163:in `call'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/transaction.rb:163:in `block (2 levels) in evaluate'
/opt/puppetlabs/puppet/lib/ruby/vendor_ruby/puppet/util.rb:386:in `block in thinmark'
/opt/puppetlabs/puppet/lib/ruby/2.1.0/benchmark.rb:294:in `realtime'
```

#### Check
You could check whether PuppetDB is working or not via this command
```
[root@katello puppet]# curl -X GET https://puppetdb.example.local:8081/pdb/meta/v1/version --cacert /etc/puppetlabs/puppet/ssl/certs/ca.pem  --cert /etc/puppetlabs/puppet/ssl/certs/$(hostname -f).pem  --key /etc/puppetlabs/puppet/ssl/private_keys/$(hostname -f).pem  --tlsv1
```
### Foreman PuppetDB Plugin
```
foreman-installer --scenario katello \
  --enable-foreman-plugin-puppetdb \
  --foreman-plugin-puppetdb-address=https://puppetdb.example.local:8081/pdb/cmd/v1 \
  --foreman-plugin-puppetdb-dashboard-address=http://puppetdb.example.local:8080/pdb/dashboard
```
