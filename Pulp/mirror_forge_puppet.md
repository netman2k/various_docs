## About
This process will mirror forge.puppet.com content. the forge site is quite slow at some time.

If you use r10k, you can set forge option with your mirror URL.

## Pulp

### Mirror forge data via Pulp
```
pulp-admin puppet repo create --repo-id=forge --description="Mirror of Puppet Forge" --display-name="Forge" --feed=http://forge.puppetlabs.com
pulp-admin puppet repo sync run --repo-id=forge
```

###Sync Scheduling
```
pulp-admin puppet repo sync schedules create -s '2017-09-21T00:00Z/P1W' --repo-id=forge
```
## Forge v3 API proxy server

The current version of Pulp does not fully support puppet forge v3 API.

There is some solution/software to solve this problem. but I want to use something that can integrate with Pulp.

Therefore I chose [Puppet Forge Server](https://github.com/unibet/puppet-forge-server/blob/master/README.md).

## Installation manual way

[https://github.com/unibet/puppet-forge-server/blob/master/README.md]

TODO

## Installation via Puppet module

[https://github.com/unibet/puppet-forge_server]

TODO