# Content Views
What can a Content View be used for?

* To stage content through environments (Dev, Test, Production).
* To filter the contents of a repository (include a package or exclude certain errata, for example).
* To have multiple snapshots of the same repository and/or puppet modules.

## Definitions
* Content View - snapshot of one or more repositories and/or puppet modules.
* Composite Content View - a Content View that contains a collection of other Content Views.
* Filter - provides finer grained control over content in a Content View. Can be used to include or exclude specific packages, package groups, or errata.
* Publishing - Content Views are ‘published’ in order to lock their contents in place. The content of the Content View is cloned and all filters applied. Publishing creates a new version of the Content View.
* Promoting - Content Views can be cloned to different Lifecycle Environments (Dev, Test, Production).

## General Workflow
First create a product and repository in the library environment and populate the repository with content (by syncing it or uploading content).
A Content Host can now register directly to library and be attached to the content therein. Updates will be available as soon as new content is synced or uploaded.

To utilize Content Views for filtering and snapshoting:

1. Create a Content View
2. Add the desired repository and/or puppet modules to the Content View
3. Optionally create one or more Filters to fine tune the content of the Content View.
4. Publish the Content View
5. Attach the Content Host to the Content View
6. Optionally promote the Content View to another environment

At this point the Content Host will no longer be getting content directly from Library, but from the Content View. Updates to library will not affect this Content Host.

Note that all of the actions below can also done with hammer, the CLI tool, and examples are given at the end of each section.

## Creating content views for yum repositories
You need to finish these before doing it:

Creating CentOS 6 Yum repositories
Creating CentOS 7 Yum repositories


I'm going to show you how to create a simple content view with no filters.

A content view will include the CentOS 6 os, updates and katello client repositories.

The other content view will include the CentOS 7 os, updates and katello client repositories.

Creating a CentOS6 Base content view
Checking repositories
You should check what repositories before adding it into a content view.

This command shows you which repositories were registered on the product, Centos 6.

[root@katello-master ~]# hammer repository list --organization-id 1 --product "CentOS 6"
---|----------------------------|----------|--------------|----------------------------------------------------------------------------
ID | NAME                       | PRODUCT  | CONTENT TYPE | URL                                                                        
---|----------------------------|----------|--------------|----------------------------------------------------------------------------
15 | updates x86_64             | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/updates/x86_64/                      
18 | storage x86_64 gluster-3.8 | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/storage/x86_64/gluster-3.8           
17 | storage x86_64 gluster-3.7 | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/storage/x86_64/gluster-3.7           
20 | sclo x86_64 sclo           | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/sclo/x86_64/sclo                     
19 | sclo x86_64 rh             | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/sclo/x86_64/rh                       
14 | os x86_64                  | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/os/x86_64/                           
21 | katello client 3.1 x86_64  | CentOS 6 | yum          | https://fedorapeople.org/groups/katello/releases/yum/3.1/client/el6/x86_64/
16 | extras x86_64              | CentOS 6 | yum          | http://centos.mirror.cdnetworks.com/6/extras/x86_64/                       
---|----------------------------|----------|--------------|----------------------------------------------------------------------------

Creating CentOS6 Base content view
You need to group these repositories which are os, updates and katello client for operating the CentOS 6 OS properly.

So I created a content view named "CentOS6 Base" as below:

hammer content-view create --name "CentOS6 Base" --description "CentOS 6 Operating System" --repository-ids 14,15,21 --organization-id 1
checking result
[root@katello-master ~]# hammer content-view list --organization-id=1 --name "CentOS6 Base"
----------------|--------------|--------------|-----------|---------------
CONTENT VIEW ID | NAME         | LABEL        | COMPOSITE | REPOSITORY IDS
----------------|--------------|--------------|-----------|---------------
4               | CentOS6 Base | CentOS6_Base |           | 14, 15, 21    
----------------|--------------|--------------|-----------|---------------
Creating a CentOS7 Base content view
Checking repositories
[root@katello-master ~]# hammer repository list --organization-id 1 --product "CentOS 7"
---|------------------------------|----------|--------------|----------------------------------------------------------------------------
ID | NAME                         | PRODUCT  | CONTENT TYPE | URL                                                                        
---|------------------------------|----------|--------------|----------------------------------------------------------------------------
3  | updates x86_64               | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/updates/x86_64/                      
6  | storage x86_64 gluster-3.8   | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/storage/x86_64/gluster-3.8/          
5  | storage x86_64 ceph-hammer   | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/storage/x86_64/ceph-hammer/          
13 | sclo x86_64 sclo             | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/sclo/x86_64/sclo/                    
12 | sclo x86_64 rh               | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/sclo/x86_64/rh/                      
4  | paas x86_64 openshift-origin | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/paas/x86_64/openshift-origin/        
7  | os x86_64                    | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/os/x86_64/                           
22 | katello client 3.1 x86_64    | CentOS 7 | yum          | https://fedorapeople.org/groups/katello/releases/yum/3.1/client/el7/x86_64/
11 | extras x86_64                | CentOS 7 | yum          | http://centos.mirror.cdnetworks.com/7/extras/x86_64/                       
---|------------------------------|----------|--------------|----------------------------------------------------------------------------

Creating CentOS7 Base content view
hammer content-view create --name "CentOS7 Base" --description "CentOS 7 Operating System" --repository-ids 3,7,22 --organization-id 1
Checking result
[root@katello-master ~]# hammer content-view list --organization-id=1 --name "CentOS7 Base"
----------------|--------------|--------------|-----------|---------------
CONTENT VIEW ID | NAME         | LABEL        | COMPOSITE | REPOSITORY IDS
----------------|--------------|--------------|-----------|---------------
5               | CentOS7 Base | CentOS7_Base |           | 3, 7, 22      
----------------|--------------|--------------|-----------|---------------
