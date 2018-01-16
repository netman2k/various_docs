# Troubleshootings

## How to stop all paused/running synchronization processes at once using "foreman-rake console"

Reference

* https://access.redhat.com/solutions/1554783
* http://api.rubyonrails.org/classes/ActiveRecord/Base.html

Sometimes synchronization processes stay in pending/paused state due to other errors in Foreman/Katello configuration. Once these errors are removed, these repositories cannot be re-synced until all locked processes are removed. Removing them one by one using Foreman WebUI (Monitor-->Tasks) is a tedious process. The following article describes how to stop all of the processes with one command.

### Run foreman-rake console
There are three states (running, paused, stopped). Stopped processes do not take up any locks. To destroy all paused processes use the following commands:

```
# foreman-rake console
```

If you get a warning as below then just run the command that recommand

```
[root@katello01 ~]# foreman-rake console
/usr/share/foreman/lib/tasks/console.rake:6: warning: already initialized constant ARGV
For some operations a user must be set, try User.current = User.first
Loading production environment (Rails 4.2.5.1)
Failed to load console gems, starting anyway
irb(main):001:0> User.current = User.first
=> #<User id: 3, login: "admin", firstname: "Admin", lastname: "User", mail: "ops-sys@cdngp.net", admin: true, last_login_on: "2016-12-20 05:37:55", auth_source_id: 1, created_at: "2016-11-29 03:29:43", updated_at: "2016-11-29 03:54:41", password_hash: "9097691f4fad1403229fb0c78dd37b797a7fbaa2", password_salt: "a194ad8825e5e6d425bcf59489a62297b3b604e0", locale: "en", avatar_hash: nil, default_organization_id: 1, default_location_id: nil, lower_login: "admin", mail_enabled: true, timezone: "">
```

### List up paused tasks
You could find paused tasks by running this command

```
ForemanTasks::Task.where(:state=>:paused)
```

**Example**

```
irb(main):003:0> ForemanTasks::Task.where(:state=>:paused)
=> #<ActiveRecord::Relation [#<ForemanTasks::Task::DynflowTask id: "5d00eaa3-7ff5-49ac-b08c-8725ad8d7edb", type: "ForemanTasks::Task::DynflowTask", label: "Actions::Katello::Repository::UploadFiles", started_at: "2016-12-20 01:40:11", ended_at: nil, state: "paused", result: "error", external_id: "4728a1ae-903a-4cc4-8031-d85fbda28d2b", parent_task_id: nil, start_at: "2016-12-20 01:40:11", start_before: nil>, #<ForemanTasks::Task::DynflowTask id: "5609eb35-6d42-4a27-abe4-20bde8ec63bb", type: "ForemanTasks::Task::DynflowTask", label: "Actions::Katello::Repository::CapsuleGenerateAndSy...", started_at: "2016-12-20 01:00:32", ended_at: nil, state: "paused", result: "error", external_id: "54cb533d-209c-4daf-8639-b8030d3f9a03", parent_task_id: nil, start_at: "2016-12-20 01:00:32", start_before: nil>, #<ForemanTasks::Task::DynflowTask id: "970eeaf7-772b-42e5-89bc-a8572b953ead", type: "ForemanTasks::Task::DynflowTask", label: "Actions::Katello::Repository::CapsuleGenerateAndSy...", started_at: "2016-12-16 01:02:42", ended_at: nil, state: "paused", result: "error", external_id: "47cc95ee-0d2f-43ad-8ef2-5094c91bbff1", parent_task_id: nil, start_at: "2016-12-16 01:02:42", start_before: nil>, #<ForemanTasks::Task::DynflowTask id: "0d10eb31-c13b-4cb4-9927-a8e6a29fb93d", type: "ForemanTasks::Task::DynflowTask", label: "Actions::Katello::Repository::CapsuleGenerateAndSy...", started_at: "2016-12-15 08:07:26", ended_at: nil, state: "paused", result: "error", external_id: "ba930695-2d16-4de8-9c14-ac09e027fb67", parent_task_id: nil, start_at: "2016-12-15 08:07:26", start_before: nil>]>
List up just specific tasks only
As a result, I set the filter to get the UploadFiles error only.

irb(main):019:0> ForemanTasks::Task.where(:state=>:paused).where(:label => "Actions::Katello::Repository::UploadFiles")
=> #<ActiveRecord::Relation [#<ForemanTasks::Task::DynflowTask id: "5d00eaa3-7ff5-49ac-b08c-8725ad8d7edb", type: "ForemanTasks::Task::DynflowTask", label: "Actions::Katello::Repository::UploadFiles", started_at: "2016-12-20 01:40:11", ended_at: nil, state: "paused", result: "error", external_id: "4728a1ae-903a-4cc4-8031-d85fbda28d2b", parent_task_id: nil, start_at: "2016-12-20 01:40:11", start_before: nil>]>
Remove specific task
I'm going to remove UploadFiles error.

irb(main):020:0> ForemanTasks::Task.where(:state=>:paused).where(:label => "Actions::Katello::Repository::UploadFiles").destroy_all
=> [#<ForemanTasks::Task::DynflowTask id: "5d00eaa3-7ff5-49ac-b08c-8725ad8d7edb", type: "ForemanTasks::Task::DynflowTask", label: "Actions::Katello::Repository::UploadFiles", started_at: "2016-12-20 01:40:11", ended_at: nil, state: "paused", result: "error", external_id: "4728a1ae-903a-4cc4-8031-d85fbda28d2b", parent_task_id: nil, start_at: "2016-12-20 01:40:11", start_before: nil>]

irb(main):021:0> ForemanTasks::Task.where(:state=>:paused).where(:label => "Actions::Katello::Repository::UploadFiles")
=> #<ActiveRecord::Relation []>
```

## How to enable logging

Reference 

* https://access.redhat.com/solutions/1155573

### Foreman/Katello
* Modify the /usr/share/foreman/config/environments/production.rb file and ensure the following line exists:
 
```
config.log_level = :debug
```

* Restart httpd service on Foreman/Katello:

```
# service httpd restart
```

* Verify that more verbose messages are shown in /var/log/foreman/production.log

### Enabling selective foreman/katello logging
Since Foreman/Katello 6.2, selective logging of katello and foreman is available. 
> Note that default debugs might not enable all areas to debug.

First, change the settings and then apply the change.

To change settings, edit /etc/foreman/settings.yaml on two places. First, set appropriate level of logging (info, debug,..):

```
:logging:
  :level: debug
```

Second, to selectively enable some loggers, add to the end:

```
:loggers:
  :ldap:
    :enabled: true
  :permissions:
    :enabled: true
  :sql:
    :enabled: true
```

It is possible to enable just some of the loggers. Note that to see logging from some area, debug logging has to be set.

Complete list of loggers with their default values is:

```
  :app:
    :enabled: true
  :ldap:
    :enabled: false
  :permissions:
    :enabled: false
  :sql:
    :enabled: false
```

To apply the change, restart Satellite service:

```
katello-service restart
```
**WARNING**
> Do not set logging to error level, otherwise some functionality can be lost - see this upstream bug for more.

## Puppet
Refer to https://docs.puppetlabs.com/references/latest/configuration.html to enable debug logging
Observe /var/log/puppet/logs for log outputs

## Pulp
http://docs.pulpproject.org/user-guide/troubleshooting.html has details on Pulp logging.
Ensure rsyslog allows debug logs to /var/log/messages or redirect such debug logs elsewhere.
In /etc/pulp/server.conf, change line:
```
# log_level: INFO
```
to:
```
log_level: DEBUG
```
Apply the change by restarting appropriate services:

```
# for i in pulp_resource_manager pulp_workers pulp_celerybeat; do service $i restart; done
```

If setting pulp to Debug, with rsyslog you may encounter a situation where many logs are discarded and missed. If that is the case, you can [disable rsyslog Rate-Limiting](https://access.redhat.com/solutions/156863) or create a new logfile called pulp and then inside this file, add the following:

```
# vi /etc/rsyslog.d/pulp.conf

:programname, startswith, "pulp" -/var/log/pulp.log
 & ~
```

Save this file and restart rsyslog and then pulp.

```
# service rsyslog restart
# for i in pulp_resource_manager pulp_workers pulp_celerybeat; do service $i restart; done
```

Check content of file /var/log/pulp.log.

## Candlepin
* Add to /etc/candlepin/candlepin.conf line:

```
log4j.logger.org.candlepin=DEBUG
```

* Restart the tomcat service (tomcat6 if RHEL6 and 
tomcat if RHEL7) : 

```
# service tomcat6 restart
```

* Verify debug output in log file /var/log/candlepin/candlepin.log
* If the candlepin logfiles are found to be to verbose with the default settings, the following can be configured in /etc/candlepin/candlepin.conf:

```
log4j.logger.org.candlepin.resource.ConsumerResource=WARN
log4j.logger.org.candlepin.resource.HypervisorResource=WARN
```

## Capsule
* Uncomment the DEBUG line in /etc/foreman-proxy/settings.yml:

```
# WARN, DEBUG, Error, Fatal, INFO, UNKNOWN
#:log_level: DEBUG
```

* Restart the foreman-proxy service :

```
# service foreman-proxy restart
```

* Observe /var/log/foreman-proxy/proxy.log

## Hammer
* Comment out the following line in /etc/hammer/cli_config.yml:

```
 :log_level: 'error'
```

* I.e. change that line to:

```
# :log_level: 'error'
```

* Watch for debug output in log: ~/.foreman/log/hammer.log (the log directory is adjustible in the cli_config.yml file)

