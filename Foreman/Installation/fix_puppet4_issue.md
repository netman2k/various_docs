Fixing dependency problem with Puppet 4 (or PuppetServer 2.x)
I figured out the foreman-installer needs puppet-string gem to work properly with the kafo. If you do not install gem before running installer, the installer will not work after upgrading to Puppet 4 (or puppetserver 2.6).

Also the puppetlabs/strings puppet module is needed

**Foreman-installer error**
```
[root@foreman01 ~]# foreman-installer --help
/usr/share/gems/gems/kafo-1.0.5/lib/kafo/puppet_module.rb:69:in `parse': No Puppet module parser is installed and no cache of the file /usr/share/katello-installer-base/modules/candlepin/manifests/init.pp is available. Please check debug logs and install optional dependencies for the parser. (Kafo::ParserError)
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/configuration.rb:92:in `block in modules'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/configuration.rb:92:in `map'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/configuration.rb:92:in `modules'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/configuration.rb:193:in `params'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/configuration.rb:203:in `preset_defaults_from_puppet'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/kafo_configure.rb:270:in `set_parameters'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/kafo_configure.rb:99:in `initialize'
    from /usr/share/gems/gems/clamp-1.0.0/lib/clamp/command.rb:133:in `new'
    from /usr/share/gems/gems/clamp-1.0.0/lib/clamp/command.rb:133:in `run'
    from /usr/share/gems/gems/kafo-1.0.5/lib/kafo/kafo_configure.rb:154:in `run'
    from /sbin/foreman-installer:8:in `<main>'
```

```
/opt/puppetlabs/puppet/bin/gem install yard
puppet module install puppetlabs/strings
```
