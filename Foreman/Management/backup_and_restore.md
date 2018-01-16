# Backup and Restore
## Backup
In the following sections, these assumptions are being made with respect to making the backup:

* /var/backups/katello will be used as the target for backups
* All commands are executed as root

### Option One: Offline repositories backup
By default, the whole Katello instance will be turned off completely for the entire backup.

```
# katello-backup /var/backups/katello
```

```
[root@katello-master var]# katello-backup /var/backups/katello
Starting backup: 2016-08-19 13:21:38 +0900
Creating backup folder /var/backups/katello
Redirecting to /bin/systemctl stop  foreman-tasks.service
Redirecting to /bin/systemctl stop  httpd.service
Redirecting to /bin/systemctl stop  pulp_workers.service
Redirecting to /bin/systemctl stop  pulp_resource_manager.service
Redirecting to /bin/systemctl stop  pulp_celerybeat.service
Redirecting to /bin/systemctl stop  foreman-proxy.service
Redirecting to /bin/systemctl stop  tomcat.service
Redirecting to /bin/systemctl stop  qdrouterd.service
Redirecting to /bin/systemctl stop  qpidd.service
Redirecting to /bin/systemctl stop  postgresql.service
Redirecting to /bin/systemctl stop  mongod.service
Backing up config files... 
tar: Removing leading `/' from member names
tar: Removing leading `/' from hard link targets
Done.
Backing up postgres db... 
tar: Removing leading `/' from member names
Done.
Backing up mongo db... 
tar: Removing leading `/' from member names
Done.
Backing up Pulp data... 
tar: Removing leading `/' from member names
Done.
Redirecting to /bin/systemctl start  mongod.service
Redirecting to /bin/systemctl start  postgresql.service
Redirecting to /bin/systemctl start  qpidd.service
Redirecting to /bin/systemctl start  qdrouterd.service
Redirecting to /bin/systemctl start  tomcat.service
Redirecting to /bin/systemctl start  foreman-proxy.service
Redirecting to /bin/systemctl start  pulp_celerybeat.service
Redirecting to /bin/systemctl start  pulp_resource_manager.service
Redirecting to /bin/systemctl start  pulp_workers.service
Redirecting to /bin/systemctl start  httpd.service
Redirecting to /bin/systemctl start  foreman-tasks.service
Done with backup: 2016-08-19 13:39:46 +0900
**** BACKUP Complete, contents can be found in: /var/backups/katello ****
```

### Option Two: Online repositories backup
Backing up the repositories can take an extensive amount of time. You can perform a backup while online. In order for this procedure to succeed, you must not change or update the repositories database until the backup procedure is complete. Thus, you must avoid publishing, adding, or deleting content views, promoting content view versions, adding, changing, or deleting sync-plans, and adding, deleting, or syncing repositories during this time. To perform an online-backup of the repositories, run:

```
# katello-backup --online-backup /var/backups/katello
```

### Option Three: Skip repositories backup
There may be situations in which you want to see a system without its repository information. You can skip backing up the Pulp database with the following option:

```
# katello-backup --skip-pulp /var/backups/katello
```

> Please note you would not be able to restore a Katello instance from a directory where the Pulp database was skipped.

### Final check-up
After a successful backup, the backup directory should have the following files:

```
# ls /var/backups/katello 
config_files.tar.gz 
mongo_data.tar.gz 
pgsql_data.tar.gz 
```

Additionally, if you ran the backup without skipping the Pulp database, you will see the additional file:

* pulp_data.tar 

Katello instance should be up and running.

## Restore
All the following commands are executed under root system account.

> Please note only backups that include the Pulp database can be restored. To verify that your backup directory is usable, make sure it has the following files:

```
# ls /var/backups/katello  
config_files.tar.gz 
mongo_data.tar.gz 
pgsql_data.tar.gz 
pulp_data.tar 
```

Once verified, simply run:

```
# katello-restore /var/backups/katello
```

This command will require verification in order to proceed, as the method will destruct all databases before restoring them. Once the procedure is finished, all processes will be online, and all databases and system configuration will be reverted to the state and the time of the backup.

Check log files for errors, such as **/var/log/foreman/production.log** and **/var/log/messages**.