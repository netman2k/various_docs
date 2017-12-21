
# FreeBSD Commands

## Initializing disks
In order to initialize a new disk, follow the below commands

### Get a new disk information
You can get information about the new disk via this command.
```
diskinfo -v daX
```

```
root@freenas-sto2:~ # diskinfo -v da6
da6
	512         	# sectorsize
	1999844147200	# mediasize in bytes (1.8T)
	3905945600  	# mediasize in sectors
	0           	# stripesize
	0           	# stripeoffset
	243133      	# Cylinders according to firmware.
	255         	# Heads according to firmware.
	63          	# Sectors according to firmware.
	004d172b0d7ed39e21007777faa0a38c	# Disk ident.
	Not_Zoned   	# Zone Mode
```	

### Partitioning a disk
```
gpart create -s gpt daX
gpart add -t freebsd-zfs daX
```
```
root@freenas-sto2:~ # gpart create -s gpt da6
da6 created
root@freenas-sto2:~ # gpart add -t freebsd-zfs da6
da6p1 added
```




# zpool command

## Create a striped mirrored(aka. RAID 10) pool
```
zpool create NAME mirror VDEV1 VDEV2 mirror VDEV3 VDEV4
```
Example
```
root@freenas:~ # zpool create tank mirror da1p1 da2p1 mirror da3p1 da4p1 mirror da5p1 da6p1
root@freenas:~ # zpool status
  pool: freenas-boot
 state: ONLINE
  scan: none requested
config:
	NAME        STATE     READ WRITE CKSUM
	freenas-boot  ONLINE       0     0     0
	  da0p2     ONLINE       0     0     0
errors: No known data errors
  pool: tank
 state: ONLINE
  scan: none requested
config:
	NAME        STATE     READ WRITE CKSUM
	tank        ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    da1p1   ONLINE       0     0     0
	    da2p1   ONLINE       0     0     0
	  mirror-1  ONLINE       0     0     0
	    da3p1   ONLINE       0     0     0
	    da4p1   ONLINE       0     0     0
	  mirror-2  ONLINE       0     0     0
	    da5p1   ONLINE       0     0     0
	    da6p1   ONLINE       0     0     0
errors: No known data errors
```

### Adding dedicate log device(ZIL:zfs intent log)
#### Check current pool
```
root@freenas-sto1:~ # zpool status tank
  pool: tank
 state: ONLINE
  scan: none requested
config:
	NAME                                            STATE     READ WRITE CKSUM
	tank                                            ONLINE       0     0     0
	  raidz1-0                                      ONLINE       0     0     0
	    gptid/439e3904-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/47027a5f-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4a6cfdd7-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4dd7ce18-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/5144f20e-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
```
#### Get GPT ID
```
root@freenas-sto1:~ # glabel list da6p1
Geom name: da6p1
Providers:
1. Name: gptid/526bb368-b96b-11e7-b021-44a8420c7b55
   Mediasize: 400088367104 (373G)
   Sectorsize: 512
   Stripesize: 4096
   Stripeoffset: 0
   Mode: r1w1e1
   secoffset: 0
   offset: 0
   seclength: 781422592
   length: 400088367104
   index: 0
Consumers:
1. Name: da6p1
   Mediasize: 400088367104 (373G)
   Sectorsize: 512
   Stripesize: 4096
   Stripeoffset: 0
   Mode: r1w1e2

root@freenas-sto1:~ # glabel list da7p1
Geom name: da7p1
Providers:
1. Name: gptid/53f08f5f-b96b-11e7-b021-44a8420c7b55
   Mediasize: 400088367104 (373G)
   Sectorsize: 512
   Stripesize: 4096
   Stripeoffset: 0
   Mode: r1w1e1
   secoffset: 0
   offset: 0
   seclength: 781422592
   length: 400088367104
   index: 0
Consumers:
1. Name: da7p1
   Mediasize: 400088367104 (373G)
   Sectorsize: 512
   Stripesize: 4096
   Stripeoffset: 0
   Mode: r1w1e2
```
#### Adding logs device
```
root@freenas-sto1:~ # zpool add tank log gptid/526bb368-b96b-11e7-b021-44a8420c7b55
```
#### Check the result
```
root@freenas-sto1:~ # zpool status
  pool: freenas-boot
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Thu Nov  2 03:45:50 2017
config:
	NAME        STATE     READ WRITE CKSUM
	freenas-boot  ONLINE       0     0     0
	  da0p2     ONLINE       0     0     0
errors: No known data errors
  pool: tank
 state: ONLINE
  scan: none requested
config:
	NAME                                            STATE     READ WRITE CKSUM
	tank                                            ONLINE       0     0     0
	  raidz1-0                                      ONLINE       0     0     0
	    gptid/439e3904-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/47027a5f-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4a6cfdd7-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4dd7ce18-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/5144f20e-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	logs
	  gptid/526bb368-b96b-11e7-b021-44a8420c7b55    ONLINE       0     0     0
errors: No known data errors
```

#### Mirroring log device
I want to add more disk to mirroring log. to solve this problem, I need to attach another disk to the current log disk.
```
root@freenas-sto1:~ # zpool attach tank gptid/526bb368-b96b-11e7-b021-44a8420c7b55 gptid/53f08f5f-b96b-11e7-b021-44a8420c7b55
```
#### Check the result again
```
root@freenas-sto1:~ # zpool status
  pool: freenas-boot
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Thu Nov  2 03:45:50 2017
config:
	NAME        STATE     READ WRITE CKSUM
	freenas-boot  ONLINE       0     0     0
	  da0p2     ONLINE       0     0     0
errors: No known data errors
  pool: tank
 state: ONLINE
  scan: resilvered 0 in 0h0m with 0 errors on Wed Nov  8 17:56:56 2017
config:
	NAME                                            STATE     READ WRITE CKSUM
	tank                                            ONLINE       0     0     0
	  raidz1-0                                      ONLINE       0     0     0
	    gptid/439e3904-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/47027a5f-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4a6cfdd7-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4dd7ce18-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/5144f20e-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	logs
	  mirror-1                                      ONLINE       0     0     0
	    gptid/526bb368-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/53f08f5f-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
errors: No known data errors
```

### Replacing the cache(L2ARC) disk
I tried to remove the cache from the pool, tank due to occasion SMART error.

I followed these steps to remove the cache.

#### Check SMART
As you can see the below result,  the Non-medium error count is too high: 4210 compared to the normal disk result.

I decided that I should replace this disk right now before going to bad more.
```
root@freenas:~ # smartctl -a /dev/da6
smartctl 6.5 2016-05-07 r4318 [FreeBSD 11.0-STABLE amd64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor:               SanDisk
Product:              LT0400WM
Revision:             D40Z
Compliance:           SPC-4
User Capacity:        400,088,457,216 bytes [400 GB]
Logical block size:   512 bytes
LU is resource provisioned, LBPRZ=1
Rotation Rate:        Solid State Device
Form Factor:          2.5 inches
Logical Unit id:      0x5001e82002839f1c
Serial number:        42180380
Device type:          disk
Transport protocol:   SAS (SPL-3)
Local Time is:        Wed Nov  8 11:14:05 2017 KST
SMART support is:     Available - device has SMART capability.
SMART support is:     Enabled
Temperature Warning:  Disabled or Not Supported

=== START OF READ SMART DATA SECTION ===
SMART Health Status: OK

Percentage used endurance indicator: 0%
Current Drive Temperature:     36 C
Drive Trip Temperature:        60 C

Manufactured in week 33 of year 2015
Specified cycle count over device lifetime:  100000
Accumulated start-stop cycles:  33
defect list format 6 unknown
Elements in grown defect list: 0

Error counter log:
           Errors Corrected by           Total   Correction     Gigabytes    Total
               ECC          rereads/    errors   algorithm      processed    uncorrected
           fast | delayed   rewrites  corrected  invocations   [10^9 bytes]  errors
read:          0        0         0         0          0       1244.405           0
write:         0        0         0         0          0       1686.994           0
verify:        0        0         0         0          0        400.093           0

Non-medium error count:       25

SMART Self-test log
Num  Test              Status                 segment  LifeTime  LBA_first_err [SK ASC ASQ]
     Description                              number   (hours)
# 1  Background short  Completed                  48       3                 - [-   -    -]
# 2  Background long   Completed                  48       2                 - [-   -    -]
# 3  Background short  Completed                  48       1                 - [-   -    -]

Long (extended) Self Test duration: 1800 seconds [30.0 minutes]
```
#### Check bad disk
```
root@freenas:~ # smartctl -a /dev/da7
smartctl 6.5 2016-05-07 r4318 [FreeBSD 11.0-STABLE amd64] (local build)
Copyright (C) 2002-16, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Vendor:               SanDisk
Product:              LT0400WM
Revision:             D40Z
Compliance:           SPC-4
User Capacity:        400,088,457,216 bytes [400 GB]
Logical block size:   512 bytes
LU is resource provisioned, LBPRZ=1
Rotation Rate:        Solid State Device
Form Factor:          2.5 inches
Logical Unit id:      0x5001e8200283a2c8
Serial number:        42181320
Device type:          disk
Transport protocol:   SAS (SPL-3)
Local Time is:        Wed Nov  8 11:07:12 2017 KST
SMART support is:     Available - device has SMART capability.
SMART support is:     Enabled
Temperature Warning:  Disabled or Not Supported

=== START OF READ SMART DATA SECTION ===
SMART Health Status: OK

Percentage used endurance indicator: 0%
Current Drive Temperature:     36 C
Drive Trip Temperature:        60 C

Manufactured in week 33 of year 2015
Specified cycle count over device lifetime:  100000
Accumulated start-stop cycles:  33
defect list format 6 unknown
Elements in grown defect list: 0

Error counter log:
           Errors Corrected by           Total   Correction     Gigabytes    Total
               ECC          rereads/    errors   algorithm      processed    uncorrected
           fast | delayed   rewrites  corrected  invocations   [10^9 bytes]  errors
read:          0        0         0         0          0        399.596           0
write:         0        0         0         0          0         13.949           0
verify:        0        0         0         0          0       2800.619           0

Non-medium error count:     4210

SMART Self-test log
Num  Test              Status                 segment  LifeTime  LBA_first_err [SK ASC ASQ]
     Description                              number   (hours)
# 1  Background long   Completed                  48    8229                 - [-   -    -]
# 2  Background short  Completed                  48       3                 - [-   -    -]
# 3  Background long   Completed                  48       2                 - [-   -    -]
# 4  Background short  Completed                  48       1                 - [-   -    -]

Long (extended) Self Test duration: 1800 seconds [30.0 minutes]
```

#### Check pool status
```
root@freenas:~ # zpool status
  pool: freenas-boot
 state: ONLINE
  scan: scrub repaired 0 in 0h0m with 0 errors on Thu Nov  2 03:45:50 2017
config:
	NAME        STATE     READ WRITE CKSUM
	freenas-boot  ONLINE       0     0     0
	  da0p2     ONLINE       0     0     0
errors: No known data errors
  pool: tank
 state: ONLINE
  scan: none requested
config:
	NAME                                            STATE     READ WRITE CKSUM
	tank                                            ONLINE       0     0     0
	  raidz1-0                                      ONLINE       0     0     0
	    gptid/439e3904-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/47027a5f-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4a6cfdd7-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/4dd7ce18-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	    gptid/5144f20e-b96b-11e7-b021-44a8420c7b55  ONLINE       0     0     0
	logs
	  gptid/526bb368-b96b-11e7-b021-44a8420c7b55    ONLINE       0     0     0
	cache
	  gptid/53f08f5f-b96b-11e7-b021-44a8420c7b55    ONLINE       0     0     0
errors: No known data errors
```

#### Remove the cache
```
zpool remove tank gptid/53f08f5f-b96b-11e7-b021-44a8420c7b55 
```
