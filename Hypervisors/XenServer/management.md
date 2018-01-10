# XenServer Management notes

## Patch via CLI
[http://www.xenserver.org/overview-xenserver-open-source-virtualization/download.html]
As you know you can patch your XenServer via XenCenter, but it consumes time to complete
I prefer using CLI method

### Upload
```
C:\Program Files (x86)\Citrix\XenCenter>xe -s 10.40.201.152 -u root -pw *******  patch-upload file-name=\Users\daehyung.lee\Downloads\XenServer\XS62E002\XS62E002.xsupdate
59128f15-92cd-4dd9-8fbe-a0115d1b07a2
```
### Apply(in manual way)
```
C:\Program Files (x86)\Citrix\XenCenter>xe -s 10.40.201.152 -u root -pw cdnadmin patch-apply uuid=59128f15-92cd-4dd9-8fbe-a0115d1b07a2 host-uuid=cb368b35-9c77-42de-9322-e23c143f1156
```
UUID is a number that was shown after patch uploading.
host-uuid is a number of the host server.
You should extract patch file before doing it

> You cannot update the XenServer by XenCenter if you are using a unlicensed version. You should use CLI to accomplish that
> [http://gallery.technet.microsoft.com/scriptcenter/Citrix-XenServer-3d276529]

### Utility script
I created this scripts and copy any patching file on a fileserver.
When I need to patch XenServer, I mount a fileserver via NFS and run the below script
Then this script will patch in order.
```
#!/bin/bash
zips=($(ls -d XS*.zip))
for zip in "${zips[@]}";
do 
  unzip -n $zip
done

upload_files=($(ls -d *.xsupdate))
host_uuid=$(xe host-list |grep uuid | awk '{print $5}')
for file in "${upload_files[@]}";
do
  uuid=$(xe patch-upload file-name=$file 2>&1 | grep "uuid:"| awk '{print $2}')
  xe patch-apply host-uuid=$host_uuid uuid=$uuid
done

```

## Solving no management interface

You can bring your interface back, when you get a message about no management interface after reboot or boot your system by any disaster.

### Disabling HA on a pool, If you have set it
```
xe host-emergency-ha-disable Force=true
(or) 
xe pool-ha-disable
```
### Make a new master
You need to choose the one of any server and run below command:
```
xe pool-emergency-transition-to-master
```
### Recover slaves (on a new master)
```
xe pool-recover-slaves
```
Probably, you can connect your xenserver with XenCenter now.



