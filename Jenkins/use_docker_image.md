[Reference](https://hub.docker.com/_/jenkins/)

## Prepare your Jenkins home
```
mkdir /jenkins_home
parted /dev/sdb mklabel msdos
parted /dev/sdb mkpart primary 1 100%
mkfs -t xfs -L JENKINS_HOME /dev/sdb1

echo "LABEL=JENKINS_HOME /jenkins_home xfs default 0 0" >> /etc/fstab
```
## Test run container
```
docker run -u 1000 --dns 8.8.8.8 -p 8080:8080 -p 50000:50000 -v /jenkins_home:/var/jenkins_home -d --restart unless-stopped jenkins
```
> Ensure that /jenkins_home is accessible by the jenkins user in container (jenkins user - uid 1000) or use -u some_other_user parameter with docker run.

## Set TLS
TODO
