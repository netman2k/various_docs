# R10K with PuppetServer
R10k provides a general purpose toolset for deploying Puppet environments and modules. It implements the Puppetfile format and provides a native implementation of Puppet dynamic environments.

Environments are managed through Git branches and dependencies and modules can be installed via a Puppetfile.

## Installing
### Creating an unprivileged account

```
useradd -s /sbin/nologin -m r10k
```

> Not need to set password on r10k account actually.

### Installing R10K
```
gem install r10k --no-rdoc --no-ri
```

### Configuring R10K

```
mkdir -p /etc/puppetlabs/r10k
```

#### Creating a configuration file - r10k.yaml
Edit /etc/puppetlabs/r10k/r10k.yaml

*A sample of r10k configuration file*
```
---
# The location to use for storing cached Git repos
cachedir: '/var/cache/r10k'

# A list of git repositories to create
sources:
  # This will clone the git repository and instantiate an environment per
  # branch in /etc/puppet/environments
  dev:
    remote: 'https://github.com/netman2k/puppet-repository.git'
    basedir: '/etc/puppet/environments'
```

#### Configuring Hiera
Create a symlink to the Hiera config in the Puppet directory
```
ln -s /etc/hiera.yaml /etc/puppet/hiera.yaml
```

#### Set directory permissions
```
mkdir -p /var/cache/r10k
chgrp r10k /var/cache/r10k
chmod 775 /var/cache/r10k
chgrp r10k /etc/puppet/environments
chmod 775 /etc/puppet/environments
```

### Using R10K
```
su -s /bin/bash r10k -
r10k deploy environment --puppetfile
```