# Upgrade

You need to follow these steps to upgrade your Katello.

## Pre-upgrade considerations
Before you upgrade, you need to run the upgrade check script that will check for any active tasks, your version of Katello, and if there are any content hosts that will be deleted (see below)

To run the script:

```
foreman-rake katello:upgrade_check
```

In addition to the information on the screen, the command will print the number of Content Hosts that will be deleted upon upgrade and will generate a csv with the full list including the uuid, name, last checkin date, and a reason for their deletion.

## Content Hosts
Katello 3.0 unifies Host and Content Host objects to provide a more unified experience. As a result, it is no longer allowed to have more than one Content Host with the same FQDN. This matches the requirement that is already imposed on the Foreman Host object. Upon upgrade to Katello 3.0 any Hosts that violate one of three rules will be deleted:

* More than one Content Host cannot exist with the same FQDN (all but the most recently registered will be deleted upon upgrade)
* Content Hosts must have a reported FQDN
* If a Host and a Content Host exist with the same FQDN, they must be within the same Organization.
 

This report is automatically run as part of the katello:upgrade_check script. However, if you want to get just the content host info, you can run:

```
foreman-rake katello:preupgrade_content_host_check
```

### Step 1 - Backup
If Katello is running on a Virtual Machine, we recommend taking a snapshot prior to upgrading. Otherwise, take a backup of the relevant databases.

### Step 2 - Operating System
Ensure your operating system is fully up-to-date:

```
# yum -y update 
```

> NOTE: If kernel packages are updated here (e.g. 
upgrading el 6.6 to 6.7), you must reboot and ensure the new kernel and SELinux policy is loaded before upgrading Katello.

### Step 3 - Repositories
Update the Foreman and Katello release packages:

**RHEL6 / CentOS 6:**

```
# yum update -y http://fedorapeople.org/groups/katello/releases/yum/3.1/katello/RHEL/6Server/x86_64/katello-repos-latest.rpm 
# yum update -y http://yum.theforeman.org/1.12/el6/x86_64/foreman-release.rpm 
```

**RHEL7 / CentOS 7:**

```
# yum update -y http://fedorapeople.org/groups/katello/releases/yum/3.1/katello/RHEL/7Server/x86_64/katello-repos-latest.rpm 
# yum update -y http://yum.theforeman.org/1.12/el7/x86_64/foreman-release.rpm 
```

### Step 4 - Update Packages
Clean the yum cache

```
# yum clean all 
```

Update the required packages:

```
# yum -y update 
```

### Step 5 - Run Installer
The installer with the –upgrade flag will run the right database migrations for all component services, as well as adjusting the configuration to reflect what’s new in Katello 3.1

```
# foreman-installer --scenario katello --upgrade 
```

Congratulations! You have now successfully upgraded your Katello For a rundown of what was added, please see release notes.!

If for any reason, the above steps failed, please review /var/log/foreman-installer/katello.log – if any of the “Upgrade step” tasks failed, you may try to run them manually below to aid in troubleshooting.

