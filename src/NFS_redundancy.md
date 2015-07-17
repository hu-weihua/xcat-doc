<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Mini-design for External NFS Support with AIX stateless/statelite and Linux statelite](#mini-design-for-external-nfs-support-with-aix-statelessstatelite-and-linux-statelite)
- [1\. Architectural Diagram](#1%5C-architectural-diagram)
- [2\. External Interface](#2%5C-external-interface)
- [2.1 AIX non-shared_root stateless](#21-aix-non-shared_root-stateless)
- [2.2 AIX shared_root stateless](#22-aix-shared_root-stateless)
- [2.3 AIX statelite](#23-aix-statelite)
- [2.4 Linux statelite](#24-linux-statelite)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

{{:Design Warning}} 


## Mini-design for External NFS Support with AIX stateless/statelite and Linux statelite

xCAT AIX stateless/statelite and Linux statelite compute nodes need to mount NFS directories from the service node, each service node can serve hundreds of compute nodes, if the NFS service on the service node or the service node itself runs into problem, all the compute nodes served by this service node will be taken down immediately, so we have to consider providing the redundant NFS service for the compute nodes. 

The reason we are focusing on NFS and not all of these other services on the service nodes, is because NFS is the only one that will take the nodes down immediately and force a reboot. For most of the other services, the administrator will have a little while to run the utility to switch the nodes to another service node. For dhcp, the lease time can be made longer. For dns, they can replicate /etc/hosts if they want. 

There are a lot of ways to achieve the HA NFS for the stateless/statelite compute nodes, from xCAT perspective, we will focus on two methods: 

1) External NFS server: the external NFS server is setup with HA capability, for example, SONAS or GPFS CNFS, and the required NFS exports for the stateless/statelite image are also configured on the external NFS server, the stateless/statelite compute nodes will mount the NFS exports from the external NFS server instead of their management node or service nodes, so if any of the service node fails, it will not take the nodes down immediately. 

2) Setup HA NFS server on service nodes: Use any existing HA product such as IBM Tivoli System Automation to setup HA NFS server between service nodes, the HA NFS service will be provided through a service IP address, the service IP address will be failovered if the current primary server goes down, the stateless/statelite compute nodes will mount the NFS exports through the service IP address. 

This documentation describes external NFS support with AIX stateless/statelite and Linux statelite. The configuration of HA NFS on service nodes will be addressed in a separate documentation. 

## 1\. Architectural Diagram

The core idea of this external NFS server support is to redirect all the NFS mounts on the stateless/statelite nodes to an external NFS server, the external NFS server can be any configuration as long as it could provide HA NFS capability and the performance is not a problem when thousands of NFS mounts come from all the stateless/statelite nodes. 

The diagram below illustrates the architectural structure of the external NFS server support, all the compute nodes will mount NFS directories only from the external HA NFS server, the service nodes will continue to provide the other network services such as TFTP, DHCP to the compute nodes. [[img src=ExternalNFSServer.jpg]] 

When any of the service node fails, the compute nodes served by it will not be affected immediately, the applications can continue run on the compute nodes. But the administrator still needs to recover the failed service nodes very soon, because the service nodes are still providing network services to the compute nodes, for example, the DHCP lease expire before the service node is recovered will cause problem. 

## 2\. External Interface

The existing node attribute “nfsserver” is used to specify the external nfs server, the procedure needed for external NFS server support includes: 

**1\. Set the node attribute “nfsserver” to hostname of the external NFS server before running mkdsklsnode/nodeset.**

**2\. After the mkdsklsnode/nodeset command is run:**
    
       i. (Optional) Verify the operating system image and the other settings by booting up one of the nodes.
    
    
       ii. Copy the required operating system image files from the service node(hierarchy cluster) or management node(non-hierarchy)to the external NFS server.
    

When copying the operating system image files, the files modes should be preserved, thus you should specify the “preserve” option with the remote copy or remotesynchronization commands, take rsync as an example, the “-az” should be the correct option. Of course, any backup/restore commands such as tar or zip can be used to copy the operating system image files. 

To avoid confusion for both operating system deployment tools(NIM, kickstart and autoyast) and the users, it is required that the operating system image directories structure should be the same on the service nodes and the external NFS server, for example, the SPOT of an AIX image is under directory /install/nim/spot/aix71/usr on the service nodes, then the SPOT should also be copied to the directory /install/nim/spot/aix71/usr on the external NFS server. 
    
       iii. Setup the NFS exports on the external NFS server. 
    

Please be aware that the NFS export options for the AIX image directories on the external NFS server should be the same with the ones on the service nodes, a simple way is to copy the /etc/exports from the service nodes to the external NFS server. 

If the diskless nodes are running AIX and the external NFS server runs Linux, when setting up NFS exports on the external NFS server, the “insecure” exports option is needed to make the AIX be able to mount the Linux NFS exports. 
    
       iv. If any changes are made on the management node or service nodes and the mkdsklsnode/nodeset command needs to be run again, the operating system image files need to synchronized to the external NFS server. 
    

The operating system image files are different for AIX non-shared_root and shared_root configuration; there are some additional steps with the statelite support, the following sections will cover the procedure for AIX non-shared_root stateless, AIX shared_root statelss, AIX statelite and Linux statelite. 

## 2.1 AIX non-shared_root stateless

The following operating system image files need to be copied to the external NFS server for the AIX non-shared_root stateless configuration: 

**1\. SPOT**

Use lsnim -l &lt;spot_name&gt; to get the location of the SPOT, for example, /install/nim/spot/71Bcosi/usr, copy the whole directory to the external NFS server. 

For one AIX stateless image, the SPOT resource is the same across all the service nodes, so only needs to copy the SPOT from one service node to the external NFS server. Here is an example: 

On one service node: 

_#rsync -az /install/nim/spot/71Bcosi/usr &lt;external_nfs_server&gt;:/install/nim/spot/71Bcosi/_

**2\. ROOT**

Use lsnim -l &lt;root_name&gt; to get the location of the root resource, for example, /install/nim/root/71Bcosi_root. 

For the non-shared_root stateless configuration, each node has a separate root resource, all the root resources are under the same directory, you can copy the directory that contains all the root resources to the external NFS server. 

Even for the same AIX image, the root resource is different between service nodes, so the root resource directories from all the service nodes are required to be copied to the external NFS server. Here is an example: 

On all the service nodes: 

_#rsync -az /install/nim/root/71Bcosi_root &lt;external_nfs_server&gt;:/install/nim/root/_

**3\. Setup NFS on the external NFS server**

Since all the root directories for all the nodes should be exported, so there will be a lot of NFS export entries on the external NFS server, a simple way is to put the content of all the /etc/exports files on all the service nodes into the /etc/exports file on the external NFS server and then run exports -a to export all the required directories. 

## 2.2 AIX shared_root stateless

**1\. SPOT**

Use lsnim -l &lt;spot_name&gt; to get the location of the SPOT, for example, /install/nim/spot/71Bcosi/usr, copy the whole directory to the external NFS server. For one AIX stateless image, the SPOT resource is the same across all the service nodes, so only needs to copy the SPOT from one service node to the external NFS server. Here is an example: 

On one service node: 

_#rsync -az /install/nim/spot/71Bsharedrootcosi/usr &lt;external_nfs_server&gt;:/install/nim/spot/71Bsharedrootcosi/_

**2\. shared_root**

Use lsnim -l &lt;shared_root_name&gt; to get the location of the root resource, for example, /install/nim/root/71Bcosi_root. For the shared_root stateless configuration, all the nodes share the same shared_root resource, but the shared_root resource will be updated for each node when running mkdsklsnode, all the updates for specific nodes are under the etc/.client_data subdirectory in the shared_root directory, so we can copy the shared_root resource from one service node to the external NFS server, and then copy all the etc/.client_data subdirectories from all the service nodes to the external NFS server. Here is an example: 

Here is an example: 

On one service node: 

_#rsync -az /install/nim/shared_root/71Bsharedrootcosi_shared_root &lt;external_nfs_server&gt;:/install/nim/shared_root/_

On all the service nodes: 

_# rsync -az /install/nim/shared_root/71Bsharedrootcosi_shared_root/etc/.client_data &lt;external_nfs_server&gt;:/install/nim/shared_root/71Bsharedrootcosi_shared_root/etc/_

**3\. Setup NFS on the external NFS server**

All the nodes will use the same shared_root directory, so only one NFS exports entry is required for the shared_root configuration, however, the NFS exports options are different on each service node, for example, the “root=” and the “access=” are used to specify the nodes list served by each servicde node, so you still need to put the content of all the /etc/exports files on all the service nodes into the /etc/exports file on the external NFS server and then run exports -a to export all the required directories. 

  
_&lt;Designer Note:&gt;_ _1\. Should the paging resource be served from the external NFS server? _ The paging space will not be used in most of the time, xCAT can configure the paging resource as sparse_paging, we also did some experiment and it turned out that the AIX diskless client can keep running normally when we remove the paging file manually after the diskless client was up and running. So the current plan is not to failover the paging resource to avoid too many NFS exports on the external NFS server.__

_2\. For now we only put the root/shared_root and the SPOT on the external NFS server, should we consider putting the other NIM resources such as dump and user defined resources to the external NFS server?_

_3\. Is it necessary to provide external interface to users to specify which NIM resources should be put on the external NFS server?''_

## 2.3 AIX statelite

AIX statelite is actually the shared_root stateless plus “overlay” for specific files or directories through the information in tables **statelite**, **litefile** and **litetree**. So the external NFS support for AIX statelite is similar with the shared_root stateless configuration, except that the considerations for the information in tables **statelite**, **litefile** and **litetree**. 

  
**statelite table**

The “statemnt” column in the statelite table specifies the persistant read/write area where a node's persistent files will be written to, the persistent files are usually critical for applications running on the nodes, so it is recommended to point the “statemnt” to the external NFS server, the variable $noderes.nfsserver can be used in the statelite table to specify that the node's attribute “nfsserver” is the external NFS server. Here is an example of the statelite table: 

_node,image,statemnt,comments,disable_

"aixcn1",,"$noderes.nfsserver:/stateliteroot",,__

Make sure you NFS export any directories that are listed in this table with read-write permission on the external NFS server before attempting to boot the nodes. 

  
**litefile table**

The litefile table specifies the directories and files for the statelite setup along with the option to use to do the setup. All the directories specified in the liefile table should also be created and set to correct permission on the external NFS server before attempting to boot the nodes. 

**litetree table**

The litetree table controls where the initial content of the files in the litefile table come from, and the long term content of the “ro” files. If any directories in this table point to the external NFS server, make sure you NFS export them on the external NFS server before attempting to boot the nodes. 

## 2.4 Linux statelite

Linux statelite nodes need to mount the rootimg from the service nodes, to provide high availability NFS capacity, the external NFS server should serve the rootimg for the Linux statelite nodes. 

1\. Before running _nodeset &lt;node_name&gt; statelite_, change the nodes attribute nfsserver to the hostname or ip address of the external NFS server. 

2\. After _nodeset &lt;node_name&gt; statelite_ is run, sync the rootimg directory to the external NFS server, here is an example: 

_rsync -az /install/netboot/rhels5.5/ppc64/compute 9.114.47.101:/install/netboot/rhels5.5/ppc64/_

The syntax and definition of the tables **statelite**, **litefile** and **litetree** is same for Linux statelite and AIX statelite, so the considerations for these three tables are the same for Linux and AIX, please refer to the "AIX Statelite" section above for the detailed procedure. 