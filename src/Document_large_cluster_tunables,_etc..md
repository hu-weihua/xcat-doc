<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Hints and Tips for Tuning Large Scale xCAT Clusters](#hints-and-tips-for-tuning-large-scale-xcat-clusters)
- [Introduction](#introduction)
- [Hierarchical xCAT Clusters](#hierarchical-xcat-clusters)
- [System Tuning Attributes](#system-tuning-attributes)
  - [Tuning ARP (Address Resolution Protocol)](#tuning-arp-address-resolution-protocol)
    - [Tuning ARP on Linux](#tuning-arp-on-linux)
    - [Tuning ARP on AIX](#tuning-arp-on-aix)
  - [Tuning AIX Network Attributes](#tuning-aix-network-attributes)
  - [Tuning AIX ulimits](#tuning-aix-ulimits)
  - [Tuning Linux ulimits](#tuning-linux-ulimits)
  - [Tuning NFS (Network File System)](#tuning-nfs-network-file-system)
  - [Tuning TCP/IP Buffer Size for RMC](#tuning-tcpip-buffer-size-for-rmc)
    - [Linux](#linux)
    - [AIX](#aix)
- [Tuning for HPC Applications](#tuning-for-hpc-applications)
  - [HPC Tuning Guide](#hpc-tuning-guide)
  - [Tuning for PE and LAPI](#tuning-for-pe-and-lapi)
    - [Linux](#linux-1)
    - [AIX](#aix-1)
- [Tuning httpd for xCAT node deployments](#tuning-httpd-for-xcat-node-deployments)
- [Tuning Values for xCAT](#tuning-values-for-xcat)
  - [Site Table Attributes](#site-table-attributes)
  - [Command Flags](#command-flags)
  - [Consolidating xCAT Database Entries](#consolidating-xcat-database-entries)
    - [Nodegroup database entries](#nodegroup-database-entries)
    - [Regular expressions in database values](#regular-expressions-in-database-values)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

{{:Design Warning}} 


# Hints and Tips for Tuning Large Scale xCAT Clusters

# Introduction

xCAT supports clusters of all sizes. This document is a collection of hints, tips, and special considerations when working with large clusters, especially clusters with more than 128 nodes. 

The information in this document should be viewed as example data only. Many of the suggestions are based on anecdotal experiences and may not apply to your particular environment. 

Updates and new information are always welcomed, and may provide valuable help and insight to other xCAT users. Please leave comments, suggestions, and additional input in the "Discussion" tab of this wiki page. 

# Hierarchical xCAT Clusters

In an xCAT cluster, the single point of control is the xCAT management node. However, in order to provide sufficient scaling and performance for large clusters, it may also be necessary to have additional servers to help handle the deployment and management of the cluster nodes. In an xCAT cluster these additional servers are referred to as service nodes. xCAT only supports these two levels of management hierarchy: the xCAT management node as the top level, and xCAT service nodes as the second level. 

A service node is managed by the xCAT management node. A compute node can be managed by either the xCAT management node or by a service node. All xCAT commands are initiated from the management node. xCAT will then internally distribute any hierarchical commands to the service nodes as required, and gather and consolidate the results. 

When does it become important to start thinking about a hierarchical cluster? There are many factors to consider. Most clusters of 128 compute nodes or less can easily be handled by a single xCAT management node. Larger non-hierarchical clusters may be feasible given enough resources on your xCAT management node and cluster management network. 

Factors: 

  * Operating System: Non-hierarchical Linux clusters can typically be larger than AIX clusters. AIX uses NIM for node deployment, so the largest non-hierarchical AIX cluster will most likely be determined by the number of nodes the NIM master on your xCAT management node can deploy. Each service node in a hierarchical AIX cluster is configured as a NIM master to the compute nodes it manages. 
  * Type of compute nodes: Will your compute nodes be stateful with its OS installed on a local disk, Linux stateless with the OS image loaded into memory, or Linux statelite or AIX stateless requiring an NFS server for the OS image? Each of these options places different burdens on the xCAT management node and service nodes. 
  * Deployment services: Several services are used during node deployment, depending on the operating system and type of compute nodes: DHCP, TFTP, bootp, HTTP, NFS, and others. Any one of these services can become a bottleneck when trying to deploy too many nodes from a single server. 
  * Management Network bandwidth: The capability of your network fabric may require you to distribute node deployment and management load across multiple subnets. 

For details on setting up a Hierarchical xCAT cluster see: 

[Setting_Up_an_AIX_Hierarchical_Cluster] 

[Setting_Up_a_Linux_Hierarchical_Cluster] 

# System Tuning Attributes

Adjusting operating system tunables can improve large scale cluster performance, avoid bottlenecks, and prevent failures. The following sections are a collection of suggestions that have been gathered from various large scale HPC clusters. You should investigate and evaluate the validity of each suggestion before applying them to your cluster. 

## Tuning ARP (Address Resolution Protocol)

Address Resolution Protocol (ARP) is a network protocol that maps a network layer protocol address to a data link layer hardware address. For example, ARP is used to resolve an IP address to the corresponding Ethernet address. 

In very large networks, the ARP table can become overloaded, which can give the appearance that xCAT is slow. 

  


### Tuning ARP on Linux

On Linux, you may see messages in your syslog /var/log/messages that you have ARP "neighbor table overflow" errors. This [web article](http://www.uwsg.iu.edu/hypermail/linux/net/0307.3/0004.html) discusses this issue, and includes some suggestions for adjusting ARP related parameters: 

  
To temporarily change the parameters on Linux, run the following from the command line: 
    
     echo "512" &gt;/proc/sys/net/ipv4/neigh/default/gc_thresh1
     echo "2048" &gt;/proc/sys/net/ipv4/neigh/default/gc_thresh2
     echo "4096" &gt;/proc/sys/net/ipv4/neigh/default/gc_thresh3
     echo "240" &gt;/proc/sys/net/ipv4/neigh/default/gc_stale_time
    

To permanently change the parameters, add the following to /etc/sysctl.conf and reboot the Linux server: 
    
     net.ipv4.conf.all.arp_filter = 1
     net.ipv4.conf.all.rp_filter = 1
     net.ipv4.neigh.default.gc_thresh1 = 512
     net.ipv4.neigh.default.gc_thresh2 = 2048
     net.ipv4.neigh.default.gc_thresh3 = 4096
     net.ipv4.neigh.default.gc_stale_time = 240
    

  


### Tuning ARP on AIX

There are several ARP tuning values that can be set on AIX. Read the discussion on "ARP Cache Tuning" in the ["AIX Performance Guide"](http://publib.boulder.ibm.com/infocenter/aix/v7r1/topic/com.ibm.aix.prftungd/doc/prftungd/prftungd_pdf.pdf)

The following are the default settings for ARP attributes in AIX 7.1: 

     arpqsize = 12 
     arpt_killc = 20 
     arptab_bsiz = 7 
     arptab_nb = 149 

With these settings, the ARP table can hold 149 buckets (arptab_nb), with 7 entries per bucket (arptab_bsiz), for a total of 1043 entries. If a server connects to more than 1043 hosts concurrently, the system will purge an entry from the ARP table and replace it with a new request. Up to 12 requests (arpqsize) can be queued while this is processing. Entries will remain in the table for 20 minutes (arpt_killc). 

Once you have determined the appropriate values for your cluster, you can change change the attributes by running the **no** command with the new values: 
    
     no -r –o arptab_nb=256
     no -r –o arptab_bsiz=8
     no -p -o arpqsize=64
    

You must reboot your server for these changes to take affect. 

## Tuning AIX Network Attributes

The following AIX attribute settings are recommended on all servers and compute nodes in large clusters: 
    
      no -p -o tcp_recvspace=524288
      no -p -o tcp_sendspace=524288
      no -p -o udp_recvspace=655360
      no -p -o udp_sendspace=65536
      no -p -o rfc1323=1
      no -p -o sb_max=8388608
    
    
      chdev -l sys0 -a maxuproc='8192'
    

## Tuning AIX ulimits

On the Management Node and the Service Nodes, change the ulimit for root. Core is optional. You must log out and back in to take affect. 
    
    ulimit -m unlimited
    ulimit -n 102400
    ulimit -d unlimited
    ulimit -f unlimited
    ulimit -s unlimited
    ulimit -t unlimited
    ulimit -u unlimited
    

Verify: 
    
    ulimit -a
    time(seconds)        unlimited
    file(blocks)         unlimited
    data(kbytes)         unlimited
    stack(kbytes)        unlimited
    memory(kbytes)       unlimited
    coredump(blocks)     unlimited
    nofiles(descriptors) 102400
    threads(per process) unlimited
    processes(per user)  unlimited
    

## Tuning Linux ulimits

SLES: 

The following memory lock and size limits are recommended for large HPC clusters. Set these values in /etc/sysconfig/ulimit: 
    
      HARDLOCKLIMIT="unlimited"
      SOFTLOCKLIMIT="unlimited"
      HARDRESIDENTLIMIT="unlimited"
      SOFTRESIDENTLIMIT="unlimited"
    

  
RedHat: 

The following memory lock limits are recommended for large HPC clusters. Set these values in /etc/security/limits.conf: 
    
      *               soft    memlock            unlimited
      *               hard    memlock            unlimited
    

## Tuning NFS (Network File System)

xCAT uses NFS in various ways: 

  * (AIX) Mounting operating system images to diskless nodes through NIM 
  * (Linux) Mounting operating system images to statelite nodes 
  * Mounting persistent directories to statelite nodes 
  * (AIX) Installing stateful nodes through NIM 
  * (Linux) Mounting the /install and /tftpboot directories from the xCAT management node to service nodes 
  * (Linux) xCAT virtualization with KVM 

Note: For Linux stateful node installs, xCAT typically uses http instead of NFS to deploy operating system packages to the nodes. 

If you are experiencing NFS performance issues, you should first verify that you do not have any network issues. Networking problems often impact NFS. 

For NFS servers that are serving the same image to a large number of nodes, ensure that the server has enough memory to hold the entire image in its NFS cache to reduce thrashing. 

For heavily loaded Linux NFS servers, increasing the NFSD count may help improve performance: 

     For SLES, edit /etc/sysconfig/nfs and set **USE_KERNEL_NFSD_NUMBER** to a higher value than the default of 8. 
     For RedHat, update /etc/init.d/nfs and change **RPCNFSDCOUNT** to a higher 

value than the default value of 8. 

Restart your NFS service to ensure that the new value takes effect. 

For servers that are handling large numbers of NFS operations, especially NFS writes, ensure that your disk subsystem is capable of processing the load. If you are running with a local SCSI drive, if possible you may want to consider moving the filesystem to an attached storage subsystem (RAID, etc.). If that is not option, it may be necessary to add additional local disk drives or create your filesystem so that it spans multiple drives to remove physical bottlenecks in your use of the disk subsystem. 

An example of creating a filesystem spanning multiple local disks on AIX: 
    
     # Create a volume group across 8 disks
     mkvg -f -y xcatvg -s 64 hdisk2 hdisk3 hdisk4 hdisk5 hdisk6 hdisk7 hdisk8 hdisk9
     # Create a logical volume for the jfs2log that spans multiple disks
     #   This is important because the jfs2log can become a bottleneck with many writes to the
     #   filesystem
     mklv -y install_log -t jfs2log -a c -S 4K xcatvg 32  hdisk2 hdisk3
     # Create a logical volume for the NFS filesystem across all the disks
     mklv -y install_lv -t jfs2 -a c -e x xcatvg 1000 hdisk4 hdisk5 hdisk6 hdisk7 hdisk8 hdisk9
     # Create the NFS filesystem
     crfs -v jfs2 -d install_lv -m /install -A yes -p rw -a agblksize=4096 -a isnapshot=no
    

  


## Tuning TCP/IP Buffer Size for RMC

If you are using RMC for cluster monitoring, you may need to increase the TCP/IP buffer sizes on your xCAT management node only to prevent incorrect node status from being returned. If the buffer size is too low for the number of nodes in the cluster, a node could be reported as down even though it can be reached using ping and the RMC subsystem on the node is active. 

### Linux

To temporarily increase the TCP/IP buffer size on Linux, run the following from the command line: 
    
     echo 524288 &gt; /proc/sys/net/core/rmem_max
     echo 262144 &gt; /proc/sys/net/core/rmem_default
    

You must also recycle RMC. Run the following from the command line: 
    
     /usr/sbin/rsct/bin/rmcctrl -k
     /usr/sbin/rsct/bin/rmcctrl -s
    

For a permanent change, add the following lines to **/etc/sysctl.conf** and reboot the Linux server: 
    
     net.core.rmem_max = 524288
     net.core.rmem_default = 262144
    

Note: To use larger values on Linux, increase both rmem_max and rmem_default in increments of 262144. For example, increase rmem_max to 786432 and rmem_default to 524288\. 

### AIX

To temporarily increase the TCP/IP buffer size on AIX, run the following from the command line: 
    
     no -o udp_recvspace=262144
    

You must also recycle RMC. Run the following from the command line: 
    
     /usr/sbin/rsct/bin/rmcctrl -k
     /usr/sbin/rsct/bin/rmcctrl -s
    

For a permanent change use the no command and the /etc/tunables/nextboot file, as follows: 

  * The no -r command sets the value changes to apply after reboot. 
  * The no -p command sets the value changes immediately and to apply after reboot. 

See the AIX documentation for detailed command usage information. Note: To use larger values on AIX, increase the udp_recvspace value in increments of 262144, not to exceed the sb_max value. 
    
     no –L udp_recvspace
     ---------------------------------------------------------------------------------
     NAME CUR DEF BOOT MIN MAX UNIT TYPE DEPENDENCIES
     ---------------------------------------------------------------------------------
     udp_recvspace 262144 262144 262144 4K 2G-1 byte C sb_max
     ---------------------------------------------------------------------------------
     no –L sb_max
     -------------------------------------------------------------------------------
     NAME CUR DEF BOOT MIN MAX UNIT TYPE DEPENDENCIES
     -------------------------------------------------------------------------------
     sb_max 1M 1M 1M 1 2G-1 byte D
     -------------------------------------------------------------------------------
    

# Tuning for HPC Applications

## HPC Tuning Guide

See "IBM Tuning Guide for High Performance Computing Applications for IBM Power 6": &lt;http://www.ibm.com/developerworks/wikis/download/attachments/137167333/Power6_optimization.pdf?version=1&gt;

This document contains recommended compiler options, discussion of using SMT (simultaneous multi-threading), running legacy executables, MPI performance optimizations, high performance libraries, and benchmark results. 

## Tuning for PE and LAPI

The Parallel Environment Installation Guide includes information on tuning for network performance. The PE documents are available here: ["Parallel Environment (PE) Library"](http://publib.boulder.ibm.com/infocenter/clresctr/vxrx/topic/com.ibm.cluster.pe.doc/pebooks.html)

For PE 5.2.1, the following documentation provides the recommendations listed in this section. Review these documents for additional settings and detailed explanations: 

["Tuning your Linux system for more efficient parallel job performance"](http://publib.boulder.ibm.com/infocenter/clresctr/vxrx/topic/com.ibm.cluster.pe521j.install.doc/am101_tysfbpjp.html)

["Running large POE jobs and IP buffer usage (PE for AIX only)"](http://publib.boulder.ibm.com/infocenter/clresctr/vxrx/topic/com.ibm.cluster.pe521j.install.doc/am101_lgpoejobs.html)

["IBM RSCT: LAPI Programming Guide"](http://publibfp.boulder.ibm.com/epubs/pdf/a2279369.pdf), see "Chapter 13. LAPI performance considerations". 

### Linux

To enable more than 8 tasks in shared memory: 
    
     echo 268435456 &gt; /proc/sys/kernel/shmmax
    

To avoid socket send and receive buffer overflow: 
    
     echo 1048576 &gt; /proc/sys/net/core/wmem_max
     echo 8388608 &gt; /proc/sys/net/core/rmem_max
     # Or, on Power Linux:
     sysctl -w net.core.wmem_max=1048576
     sysctl -w net.core.rmem_max=8388608
    

To avoid datagram reassembly failures when MP_UDP_PACKET_SIZE is greater than the MTU: 
    
     echo 1048576 &gt; /proc/sys/net/ipv4/ipfrag_low_thresh
     echo 8388608 &gt; /proc/sys/net/ipv4/ipfrag_high_thresh
    

To enable jumbo frames: 
    
     ifconfig eth&lt;x&gt; mtu 9000 up
    

The switch must be configured with jumbo frame enabled. 

To turn off interrupt coalescing on device eth0 on Power Linux: 
    
     ifdown eth0
     rmmod e1000
     insmod /lib/modules/2.6.5-7.97-pseries64/kernel/drivers/net/e1000/e1000.ko \
          InterruptThrottleRate=0,0,0
     ifconfig eth0
    

The path to e1000.ko may vary with kernel level. 

### AIX

To avoid socket send and receive buffer overflow: 
    
      no -o sb_max=8388608
    

  
To turn off interrupt coalescing on device ent0 for better latency: 
    
      ifconfig ent0 detach
      chdev -l ent0 -a intr_rate= 0
    

  
If your user applications are using large pages, see The PE 5.2.1 Operation and Use Guide: ["Using POE with AIX Large Pages"](http://publib.boulder.ibm.com/infocenter/clresctr/vxrx/topic/com.ibm.cluster.pe521j.opuse1.doc/am102_upoetech.html)

On all compute nodes: 
    
     vmo -p -o lgpg_regions=64 -o lgpg_size=16777216
    

  
If your user applications require shared memory segments to be pinned, on all compute nodes: 
    
     vmo -p -o v_pinshm=1
    

  
If you decide you would like full system dumps to aid in problem diagnosis, on all compute nodes: 
    
     chdev -l sys0 -a fullcore=true
    

  
If your user applications perform better with Simultaneous Mult-Threading turned off (this is application dependent), on all compute nodes: 
    
      smtctl -m off
    

# Tuning httpd for xCAT node deployments

For a discussion of apache2 tuning, see the external web page: ["Apache Performance Tuning"](http://httpd.apache.org/docs/2.0/misc/perf-tuning.html)

The default settings for Apache 2 on Red Hat and SLES may not allow enough simultaneous HTTP client connections to support the installation of more than about 50 nodes at a time. To enable greater scaling, the MaxClients and ServerLimit directives need to be increased from 150 to 1000. On Red Hat, change (or add) these directives in 
    
     /etc/httpd/conf/httpd.conf
    

On SLES (with Apache2), change (or add) these directives in 
    
     /etc/apache2/server-tuning.conf
    

# Tuning Values for xCAT

xCAT provides several settings that may affect the management performance in your cluster. 

## Site Table Attributes

The xCAT site table contains many global settings for your xCAT cluster. To display current values, run the command: 
    
      lsdef -t site -l
    

This will only list those attributes that have been explicitly set in your site table. Many other attributes are supported. To list all possible attribute names, a brief description of each, and default values, run: 
    
      lsdef -t site -h
    

  
You may consider changing the default settings for some of these attributes to improve the performance on a large-scale cluster or if you are experiencing timeouts or failures in these areas: 

consoleondemand&nbsp;: When set to 'yes', conserver connects and creates the console output only when the user opens the console. Default is 'no' on Linux, 'yes' on AIX. Setting this to 'no' can reduce the load conserver places on your xCAT management node. If you need this set to 'yes', you may then need to consider setting up multiple servers to run the conserver daemon, and specify the correct server on a per-node basis by setting each node's **conserver** attribute. 

  


ipmimaxp&nbsp;: The max # of processes for ipmi hw ctrl. Default is 64. 
ipmiretries&nbsp;: The # of retries to use when communicating with BMCs. Default is 3. 
ipmisdrcache&nbsp;: If set to 'no', then the xCAT IPMI support will not cache locally the target node's SDR cache to improve performance. Default is 'no'. 
ipmitimeout&nbsp;: The timeout to use when communicating with BMCs. Default is 2. 

  


nodestatus&nbsp;: If set to 'n', the nodelist.status column will not be updated during the node deployment, node discovery and power operations. Default is 'y', always update nodelist.status. Setting this to 'n' for large clusters can eliminate one node-to-server contact and one xCAT database write operation for each node during node deployment, but you will then need to determine deployment status through some other means. 

  


powerinterval&nbsp;: The time of seconds that rpower command will wait between performing action on each target object. It is used for controlling the cluster boot up speed in large clusters. Default is 0. 

  


ppcmaxp&nbsp;: The max # of processes for PPC hw ctrl. Default is 64. 
ppcretry&nbsp;: The max # of PPC hw connection attempts before failing. Default is 3. 
ppctimeout&nbsp;: The timeout, in milliseconds, to use when communicating with PPC hw. A value of '0' means use the default. Default is 60. 
maxssh&nbsp;: The max # of SSH connections at any one time to the hw ctrl point for PPC. Default is 8. 
fsptimeout&nbsp;: The timeout, in milliseconds, to use when communicating with FSPs. A value of '0' means use the default. The default value is 30. If you are experiencing connection timeouts on a heavily loaded management network, you may want to experiment with setting this to a larger value. 

  


svloglocal&nbsp;: if set to 1, syslog on the service node will not get forwarded to the mgmt node. The default is to forward all syslog messages. The tradeoff on setting this attribute is reducing network traffic and log size versus having local management node access to all system messages from across the cluster. 

  


useNmapfromMN&nbsp;: When set to yes, nodestat command should obtain the node status using nmap (if available) from the management node instead of the service node. This will improve the performance in a flat network. Default is 'no'. 

## Command Flags

In addition to the site table settings, some xCAT commands provide options for timeouts, retries, and fanouts to help improve the potential for command success under different conditions. If you are experiencing problems running any of these commands, you may wish to experiment with some of these values: 

  * [lsslp](http://xcat.sourceforge.net/man1/lsslp.1.html)
  * [xdsh](http://xcat.sourceforge.net/man1/xdsh.1.html)
  * [xdcp](http://xcat.sourceforge.net/man1/xdcp.1.html)

## Consolidating xCAT Database Entries

The xCAT database contains over 50 different tables, although for most clusters you only need entries in a small number of commonly used tables. The majority of the tables are keyed by the xCAT node name. For very large clusters, consolidating individual node entries into entries keyed by a nodegroup name can reduce the amount of information stored in the database, and make handling the data more manageable. 

### Nodegroup database entries

With the xCAT *def commands (lsdef, mkdef, chdef, rmdef), you can manipulate common attribute values for node groups using "-t group" (object type of 'group'). These will create database entries that are keyed by nodegroup name instead of individual node names. 

For example: 
    
      chdef -t group -o compute nodetype="osi,ppc" hwtype=lpar os=rhels6 arch=ppc64
    

will set many common attributes in the nodetype table for the nodegroup "compute". 

See the xCAT documentation [Node_Group_Support] for details on using node groups and nodegroup entries in the xCAT database. 

### Regular expressions in database values

Another useful feature to use in conjunction with nodegroup entries, are to have entries that include regular expressions. This is especially useful where you would normally require an individual table entry for each node because of unique data values. When those data values follow an identifiable pattern, specifying the pattern as a regular expression enables you to create a single entry for the nodegroup. See [the xCAT Database manpage](http://xcat.sourceforge.net/man5/xcatdb.5.html) for more details and for examples. 