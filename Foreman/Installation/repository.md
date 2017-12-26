## Registering Katello 3.2 repository
This repository setting will install katello 3.2, foreman 1.13 and puppetmaster 3.8
```
yum -y localinstall http://fedorapeople.org/groups/katello/releases/yum/3.3/katello/el7/x86_64/katello-repos-latest.rpm
yum -y localinstall http://yum.theforeman.org/releases/1.14/el7/x86_64/foreman-release.rpm
yum -y localinstall http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm
yum -y localinstall http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install foreman-release-scl
```
### Registering Katello 3.3 with Puppetlabs All-In-One(AIO)
This repository setting will install katello 3.3, foreman 1.14 and puppetserver 2.x (puppet 4.x)
```
yum -y localinstall http://fedorapeople.org/groups/katello/releases/yum/3.3/katello/el7/x86_64/katello-repos-latest.rpm
yum -y localinstall http://yum.theforeman.org/releases/1.14/el7/x86_64/foreman-release.rpm
yum -y localinstall https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
yum -y localinstall http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install foreman-release-scl
```
