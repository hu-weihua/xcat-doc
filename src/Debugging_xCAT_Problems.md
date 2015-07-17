<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

  - [Debugging general issues](#debugging-general-issues)
    - [Verify xcatd is running.](#verify-xcatd-is-running)
    - [xcat plugin fails to load](#xcat-plugin-fails-to-load)
    - [Verify DB access](#verify-db-access)
    - [Name resolution](#name-resolution)
    - [Node deployment problems](#node-deployment-problems)
      - [Security ssh or xcat keys/certificates problem](#security-ssh-or-xcat-keyscertificates-problem)
      - [Verify the intended pkgs are being picked up during genimage or node deployment](#verify-the-intended-pkgs-are-being-picked-up-during-genimage-or-node-deployment)
      - [Service Node specific](#service-node-specific)
    - [Provide xCAT information for development](#provide-xcat-information-for-development)
  - [Using xcatdebug to Trace xCAT Code](#using-xcatdebug-to-trace-xcat-code)
    - [Subroutine calling trace](#subroutine-calling-trace)
    - [Commented trace log](#commented-trace-log)
  - [How to debug a general OS deployment process](#how-to-debug-a-general-os-deployment-process)
    - [Network Related Issues](#network-related-issues)
    - [Setup Domain Name Resolution DNS](#setup-domain-name-resolution-dns)
    - [Setup conserver](#setup-conserver)
    - [DHCP problems](#dhcp-problems)
    - [postscripts issues](#postscripts-issues)
    - [syslog setup problems](#syslog-setup-problems)
    - [ssh key setup problem](#ssh-key-setup-problem)
    - [Otherpkgs install problems](#otherpkgs-install-problems)
    - [Syncing  files does not work](#syncing--files-does-not-work)
    - [Node status incorrect](#node-status-incorrect)
    - [updatenode problems](#updatenode-problems)
    - [updateflag.awk failure - port 3002 closed](#updateflagawk-failure---port-3002-closed)
    - [The service node related issue](#the-service-node-related-issue)
    - [Compute node booting issue](#compute-node-booting-issue)
    - [Check dhcp stack working or not](#check-dhcp-stack-working-or-not)
    - [Kernel panic during the deployment](#kernel-panic-during-the-deployment)
    - [Debug grub2 problems](#debug-grub2-problems)
- [install rhels7.1-ppc64-compute](#install-rhels71-ppc64-compute)
    - [ssh keys setup on Node do not work](#ssh-keys-setup-on-node-do-not-work)
  - [How to debug a virtual machine management issue](#how-to-debug-a-virtual-machine-management-issue)
    - [kvm problems](#kvm-problems)
      - [Change the resource of a vm](#change-the-resource-of-a-vm)
      - [Reboot the libvirtd to handle the unknown problems](#reboot-the-libvirtd-to-handle-the-unknown-problems)
  - [How to debug the discovery](#how-to-debug-the-discovery)
    - [Discovery for system x](#discovery-for-system-x)
    - [How to perform the discovery](#how-to-perform-the-discovery)
  - [How to boot up a node with debug kernel for Linux](#how-to-boot-up-a-node-with-debug-kernel-for-linux)
  - [Enable KDB for AIX Power 775 nodes](#enable-kdb-for-aix-power-775-nodes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

![](http://sourceforge.net/p/xcat/wiki/XCAT_Documentation/attachment/Official-xcat-doc.png)



## Debugging general issues

### Verify xcatd is running.

~~~~
   lsxcatd -a
   export XCATBYPASS=1 to be able to run xcat cmds without xcatd (to look at values and use the perl debugger)
   Restart xcatd with output going to the screen: service xcatd stop; /opt/xcat/sbin/xcatd -f
~~~~

### xcat plugin fails to load

if an xcatd plugin is failing to load:

~~~~
   cd /opt/xcat/lib/perl/xCAT_plugin
   perl -c -I ../ <plugin.pm>
~~~~

if the perl -c returns "syntax ok", check if the plugin.pm ends with a line "1".
When you installed xCAT, did you also use the latest xcat-dep tarball from http://sourceforge.net/projects/xcat/files/xcat-dep/2.x_Linux or http://sourceforge.net/projects/xcat/files/xcat-dep/2.x_AIX&nbsp;?

### Verify DB access

~~~~
   tabdump site
   lsxcatd -d
~~~~

Verify that some key site attributes are correct:

~~~~
   domain
   master
~~~~

Verify the [networks table](http://xcat.sourceforge.net/man5/networks.5.html) correctly describes the networks in your cluster

### Name resolution

Make sure you have name resolution for every defined ip address for the Management Node and the Service Nodes .

~~~~
   run ifconfig -a | grep inet
~~~~

Take the list of ips that are output and run getent hosts &lt;ipaddres&gt;
   make sure each comes back with a resolved hostname
If not add the ip address to /etc/hosts
   See [Cluster_Name_Resolution]

Name resolution issues often cause many problems that are difficult to debug. Some symptoms that have been seen:

  * Statefull (full-disk) installs may hang, fail, or go into "infinite" re-install loops
  * Diskless and statelite installs may hang, be extremely slow, or have problems running xCAT postscripts
  * If an /etc/resolv.conf file has an entry that is not reachable during node install or boot, this can cause long delays, timeouts, and hangs.
  * Hierarchical commands such as xdsh will fail or take a long time to run, if name resolution is not set up correctly on the Management Node and Service nodes.

### Node deployment problems

  * Display problem node attributes using lsdef
  * DHCP:
    * tail -f /var/log/messages during node deployment to track DHCP requests, etc.
    * Are there two dhcp servers running on the same subnet? Check by running this new xcat tool on any node that is connected to the installation network (can be the MN or SN): /opt/xcat/share/xcat/tools/detect_dhcpd -i &lt;network-facing-NIC&gt;
     *Is dhcp listening on the correct interfaces? Check site.dhcpinterfaces and only specify your cluster-facing network interfaces, then run 'makedhcp -n' to create a new dhcp conf file.

#### Security ssh or xcat keys/certificates problem

 *  See if ssh  &lt;node&gt; date can run w/o prompting for pw
 *  Run xdsh -K to update the ssh keys on the nodes
 *  Run updatenode -k to update keys/certificates on the servicenodes
 *  If nothing else works, run xcatconfig with appropriate flag(-k,-s,-c). See man page [xcatconfig](http://xcat.sourceforge.net/man8/xcatconfig.8.html)  for what you must do after running these commands. 
 *  Check the auditlog xcat table for denials from the policy table

#### Verify the intended pkgs are being picked up during genimage or node deployment

~~~~
   lsdef <node> \--osimage
   check image table: lsdef -t osimage <osimage>
~~~~

Verify all necessary linux services are running

~~~~
   tftp, nfs, http, dhcp, conserver, named
   Verify no file systems are full on MN or SNs
   Verify SELinux is disabled, especially on RHEL*
~~~~

#### Service Node specific

  * Verify it can access the db on the MN and that the /etc/xcat/cfgloc contents is correct
~~~~
     lsxcatd -a
~~~~

  * Make sure the xCAT versions are the same on MN and SNs
~~~~
     lsxcatd -a on MN and SN
~~~~

  * Make sure the date/time is the same between the MN and SNs

If /install is not mounted on the SN, make sure it is sync'd to all the SNs

AIX specific:
   *Is DHCP running on NIM master? Isn't this needed on the p775 EMS?

### Provide xCAT information for development

   *Run xcatsnap and send to the xCAT development team

##  Using xcatdebug to Trace xCAT Code

A new command [xcatdebug](http://xcat.sourceforge.net/man8/xcatdebug.8.html) was introcuded in 2.7 for the admin to enable the trace for the xcatd daemon and plugins.

xcatdebug offers two trace facilities to debug the xCAT:

### Subroutine calling trace

Display the calling trace for subroutine when it is called.

The flag -c is used to specify the subroutine list for subroutine calling trace, it can only work with -f. The value of -c can be a configuration file or a subroutine list.

     configuration file: a file contains multiple lines of SUBROUTINE_DEFINITION
     subroutine list:    SUBROUTINE_DEFINITION|SUBROUTINE_DEFINITION|...


SUBROUTINE_DEFINITION: is the element for the -c to specify the subroutine list.

The format of SUBROUTINE_DEFINITION: [plugin](subroutine1,subroutine2,...)

If ignoring the [plugin], the subroutines in the () should be defined in the xcatd.

       e.g. (daemonize,do_installm_service,do_udp_service)


Otherwise, the package name of the plugin should be specified.

       e.g. xCAT::Utils(isMN,Version)
       e.g. xCAT_plugin::DBobjectdefs(defls,process_request)


The trace log will be written to /var/log/xcat/subcallingtrace. The log file subcallingtrace will be backed up for each running of the xcatdebug -f enable.

Enable the subroutine calling trace for all the subroutines in the xcatd and plugin modules.

~~~~
    xcatdebug -f enable
~~~~


Enable the subroutine calling trace for the plugin_command in xcatd and defls,process_request in the xCAT_plugin::DBobjectdefs module.

~~~~
    xcatdebug -f enable -c "xCAT_plugin::DBobjectdefs(defls,process_request)|(plugin_command)"
~~~~


### Commented trace log

The trace log code is presented as comments in the code of xCAT. In general mode, it will be kept as comments. But in debug mode, it will be commented back as common code to display the trace log.

This facility offers two formats for the trace log code:

Trace section

~~~~
    ## TRACE_BEGIN
    # print "In the debug\n";
    ## TRACE_END
~~~~


Trace in a single line

~~~~
     ## TRACE_LINE print "In the trace line\n";
~~~~


Enable the commented trace log

~~~~
    xcatdebug -d enable
~~~~


This facility also can be enabled by passing the ENABLE_TRACE_CODE=1 global variable when running the xcatd.

~~~~
    ENABLE_TRACE_CODE=1 xcatd -f
~~~~


## How to debug a general OS deployment process

### Network Related Issues

   *Look at syslog on the MN

     *By default the syslog is written into the file /var/log/messages. You can tail -f it to watch new entries as your nodes deploy.

Following messages should be noticed:

  * DHCP - The dhcpd on MN/SN should get the dhcp request from node and send back the offer/ack to the node. If no dhcp request is received: 1. try to make sure the node is attempting to boot from network (not its local disk); 2. verify the dhcp server has been started. (for bootp on aix, it will be start by xinetd automatically)

  * atftp - To see whether the bootloader file xnba.kpxe/pxelinux.0/yaboot has been transferred to the node.

If the syslog postscript runs correctly for the SN/CN, then after that you will also see the syslog messages redirected from SN/CN to the MN.

Look at /etc/dhcpd.conf and /var/lib/dhcpd/dhcpd.leases
   *  /etc/dhcpd.conf (on RHEL6, this is in /etc/dhcp/dhcpd.conf) - Verify it has the correct stanza for the MN/SN's installation nic. (The dynamic rage should only be used for the discovery process.) This stanza was created by [makedhcp -n](http://xcat.sourceforge.net/man8/makedhcp.8.html) and is also affected by the [site.dhcpinterfaces](http://xcat.sourceforge.net/man5/site.5.html) attribute.
   *  /var/lib/dhcpd/dhcpd.leases - verify it has the correct section for each node. the mac and assigned IP should be included. These entries were created by [makedhcp -a](http://xcat.sourceforge.net/man8/makedhcp.8.html) (or makedhcp &lt;noderange&gt;).

The files in the /tftpboot

    * The bootloader, configuration file of bootloader, initrd and osimage are located in the /tftpboot. Make sure they have been generated successfully before the starting of installation.

    *For netboot with yaboot: the file 'yaboot' should be installed in /tftpboot. The configuration files should be genreated in the /tftpboot/etc/ with node name.
    *For netboot with pxe: the file 'pxelinux.0' should be installed in /tftpboot. The configuration files should be generated in the /tftpboot/pxelinux.cfg/ with node name.
   For netboot with xnba: the files nbfs.&lt;arch&gt;.gz and nbk.&lt;arch&gt; should be installed in /tftpboot/xcat. The configuration files should be generated in the /tftpboot/xcat/xnba/nodes/&lt;node&gt; with node name.
    *Note: If the kernel or initrd/osimage can not be loaded correctly, look in the bootloader configuration file to verify that the files IP/Path/file_names have been installed.

Look in the log of the http server

During the OS deployment, some files will be transported by http. Monitoring the http log could help for debugging.

The location of http log - For readhat: /var/log/httpd/access_log; For sles: /var/log/apache2/access_log
If you use the xnba as boot method, the kernel, image will be gotten from http.
If you failed to get xnba file and get error messages such as "Filename: http://172.20.0.11/tftpboot/xcat/xnba/nodes/cn

http://172.20.0.11/tftpboot/xcat/xnba/nodes/cn... Connection reset" . Try to check 1)if firewall is stopped. 2) if httpd is running.



For diskfull, the kickstart configuration and installed packages will be gotten from http.

The installation configuration file /install/autoinst/&lt;node&gt;

This file will be generated for diskfull installation after the running of 'nodeset &lt;node&gt; install'.

Check the following part to work out possible problems:

~~~~
       url
       packages
       pre
       post
~~~~


###Setup Domain Name Resolution DNS

The hostname of SN/CN should can be resolved from MN. And the hostname of MN/SN should can be resolved from CN.  See [Cluster_Name_Resolution]

You should can run the command 'nslookkup node' to see whether the dns setup for the node has been done. If you don't want to use dns, just run 'ping node' to see the IP of node has been worked out by ping command.


Run following command to setup DNS, see man page for options and [Cluster_Name_Resolution].

~~~~
   makedns
~~~~

###Setup conserver

Run rcons to open the console to monitor the installation process.

If 'rcons node' encountered problem, try following methods:


   Run 'makeconservercf' to add the the node into the configuration file of conserver.
   Check configuration file of conserver /etc/conserver.cf to see if node is added.

~~~~
      :For cn managed by hmc, the configuration file should be like
        #xCAT BEGIN cn CONS
        console cn {
          type exec;
          exec /opt/xcat/share/xcat/cons/hmc cn;
        }
        #xCAT END cn CONS
        #
      :For cn managed by BMC with IPMI, the configuration file should be like
        #xCAT BEGIN cn CONS
        console cn {
          type exec;
          exec /opt/xcat/share/xcat/cons/ipmi cn;
        }
        #xCAT END cn CONS
      :For virtual machine managed by kvm, the configuration file should be like
        #xCAT BEGIN cn CONS
        console cn {
          type exec;
          exec /opt/xcat/share/xcat/cons/kvm cn;
        }
        #xCAT END cn CONS
       :For cn managed by fsp, the configuration file should be like
         #xCAT BEGIN lpar1 CONS
         console lpar1 {
           type exec;
           exec /opt/xcat/share/xcat/cons/fsp lpar1;
         }
         #xCAT END lpar1 CONS

~~~~



If you node are system x node, you may hit error "@localhost: Connection refused", please check node definition and make sure your node has following attributes.

~~~~
       serialflow=hard
       serialport=0
       serialspeed=115200
~~~~




Run 'service conserver reset' to resolve the certificate key issue.

Check the validation of attributes in the tables

If you encountered some issues that do not have hit to figure out the cause, try to check the attributes in following tables.


   Table Site: master, domain, nameserver,dhcpinterfaces
   Table networks: Has the installation network defined in.
   Table noderes: netboot, tftpserver,nfsserver,installnic,primarynic
   Table nodetype: os,arch,profile

### DHCP problems
Should not have multiple dhcpd daemons in one network/vlan

If there are two or more dhcpd daemons in the installation network/vlan responding to node requests, it will cause strange issues, so you should verify that there is only one dhcp server in the network. This can be done by running a new xcat tool on any node that is connected to the installation network (can be the MN or SN):

~~~~
   /opt/xcat/share/xcat/tools/detect_dhcpd -i <network-facing-NIC>
~~~~

### postscripts issues

Login the compute node and check:

1. Is /xcatpost created and the postscripts are downloaded correctly? If the /xcatpost is not created, there might be some network problems between the compute node and management node. For SLES, check the /var/adm/autoinstall/scripts/ and /var/adm/autoinstall/logs/.

2. the content of following script files, to see whether the exported environment variable has correct value and the postscripts have been added correctly.

   Linux: /xcatpost/mypostscript, /xcatpost/mypostscript.post
   AIX: /xcatpost/myxcatpost_<nodename>

A specific postscript is not run during the installation

   Make it has been added into the postscripts.postscripts or postscripts.postbootscripts
   Make sure it has been copied to the /install/postscripts.
   Make sure the permission on the file is world readable and executable by root.

### syslog setup problems
All node syslogs are sent to the Management node by default.
As default, the syslog message from the node will be redirected to the management node. Make sure the syslog postscript has been added into the postscripts attribute.

Look into configuration file of syslog: /etc/(r)syslog.conf (rh), /etc/syslog-ng/syslog-ng.conf (sles).

### ssh key setup problem
Cannot login the node  without entering a password.

     Usually, the 'remoteshell'(linux) or 'aixremoteshell'(AIX) was not run correctly for the node. Make sure node can be resolved from management, and management node can be resolved from node.

### Otherpkgs install problems
Linux: The packages listed in the otherpkgs are not installed on the node



Make sure the path and name of otherpkgs configuration file are correct. Path: /opt/xcat/share/xcat/netboot(install)/&lt;platform&gt;/profile.&lt;os&gt;.&lt;arch&gt;.otherpkgs.pkglist
Make sure the packages have been copied to correct path. Path: /install/post/otherpkgs/&lt;os&gt;/&lt;arch&gt;/xcat

AIX: The packages listed in the installp_bundle and otherpkgs are not installed on the node



Make sure the installp_bundle or otherpkgs has been configured to the nimimage. 1. Wirte into table nimimage. 2. Put as parameter for 'nimnodeset' command.
Make sure the installp and rpm packages have been updated to the nimimage.

### Syncing  files does not work
The files listed in the synclist are not synced to the node

Make sure the synclist configuration file has correct format. Refer to the doc [Sync-ing_Config_Files_to_Nodes].


### Node status incorrect
After the installation, the 'status' attribute is not changed to 'booted'
Make sure the process 'xcatd: install monitor' has been started correctly.

~~~~
     ps -ef | grep xcatd
~~~~

### updatenode problems
   For debugging updatenode problems, see [Using_Updatenode#Debugging_Tips](Using_Updatenode/#appendix-c-debugging-tips).

### updateflag.awk failure - port 3002 closed

Check to see if the install monitor pid is running on the MN.

~~~~
    ps axf|grep -i xcatd
~~~~


If you see 'install monitor', then

~~~~
    lsof -p <pid>
~~~~


A good tool to install is NetCat:

~~~~
    nc-1.84-22.el6.ppc64
~~~~


You can stop xcatd and then run the tool and see if it can open the port

~~~~
    nc -l 3002
~~~~


Then if you can start xcatd and see it cannot open the port:

~~~~
    /opt/xcat/sbin/xcatd -f
    Starting vsftpd for vsftpd:                               [  OK  ]
    xcatd unable to open install monitor services on 3002
    Died at /opt/xcat/sbin/xcatd line 262.
    -------startMonitoring: product_names=
~~~~


### The service node related issue

     For a service node, at least the servicenode.tftpserver attribute should be set to '1', otherwise it will not be configured as a service node.

Some specific postscripts are not run

     There are several postscripts will be run for the service node: servicenode, xcatserver (only for Linux) and xcatclient (only for Linux). If you found certain of these postscript was not run, check whether the node has been added into the 'service' group. Or check whether the scirpts have been added into the 'postscripts' attribute.

The xCAT rpm packages are not installed on the service node



For linux, make sure you have used the 'service' as the profile for the node. And copy the xcat-core and xcat-dep which untared from the xCAT installation packages into the path: /install/post/otherpkgs/&lt;os&gt;/&lt;arch&gt;/xcat.
   For AIX, make sure the correct installp_bundle for service node has been added for the node. And updated the related installp or rpm packages for the nimimage.

The xcatd cannot start

     For service node, only the database which supporting the remote access can be used. The supported database: Mysql, Postgresql, DB2. The database on service node should be setup correctly to access the xCAT DB on the management node, otherwise the xcatd cannot start up correctly. Refer to the database configuration doc for the verification.

Some specific servers are not started



   Make sure the xcatd has been started.
   Make sure the the services have been set in the servicenode table.

### Compute node booting issue

   *Add drucat debug shell support

     After nodeset to the compute node, add "rdshell" and "rdinitdebug" to kernel parameters in /tftpboot/pxelinux.cfg/&lt;cn&gt;, these two parameters will make the kernel start a shell if any error happens during boot.
     So you can debug in the shell to see if the ip addresses have been configured and if the rootimage can be downloaded from MN/SN
     If any tools are not available on CN, mount a directory with toolkits from SN/MN.

     The /tftpboot/pxelinux.cfg/&lt;cn&gt; file will be something like:

  1. install rhels6.1-x86_64-kvm

DEFAULT xCAT LABEL xCAT

~~~~
    KERNEL xcat/rhels6.1/x86_64/vmlinuz
    APPEND initrd=xcat/rhels6.1/x86_64/initrd.img quiet tmpfs_size=2G  repo=http://192.168.5.84/install/rhels6.1/x86_64/ ks=http://192.168.5.84/install/autoinst/x3550xcat ksdevice=eth0 cmdline console=tty0 console=ttyS0,19200n8r rdshell rdinitdebug
     IPAPPEND 2
~~~~




###Check dhcp stack working or not

     Since the login node can boot up after changing the initrd, it is more like dhclient rpm is not installed correctly in the bad initrd, you can check the section of installing additional HFI related packages in our pLinux doc to see if there is any hints.

     A good way to debug dhcp problem is to trigger a dhcp request on the compute node and see what we can get from server. The command to trigger a dhcp request on compute node:

     In the rdshell on compute node that created by above step, check if /etc/dhclient.conf and /tmp/dhclient.eth0.lease exist. If they do, issue:

~~~~
       dhclient -4 -1 -q -cf /etc/dhclient.conf -pf /tmp/dhclient.eth0.pid -lf /tmp/dhclient.eth0.lease eth0
~~~~


     If we are not getting too many syslog entries  on dhcp server, we can trigger a dhcp request and use tcpdump -w option on service node or management node to log all the network communication and parse the network log for analysis.

     Also if it is initrd prolbem on System P servers, you can extract the two initrd and compare the dhcp/dhclient related binaries size directly to see if there is any hint.

     The way to extract initrd:

~~~~
       gunzip initrd-stateless.gz
       mkdir 1
       cd 1
       cpio -id <node> serialflow=
~~~~


    2\. Unset all the serial console related attributes:

~~~~
      chdef <node> serialport= serialspeed= serialflow=
~~~~


    3\. Revert the console parameter order:
    Get in the pxe/xnba configuration file (/tftpboot/xcat/xnba/nodes/&lt;node&gt;, /tftpboot/pxelinux.cfg/&lt;node&gt;), revert the order of console parameters:

~~~~
      Change from
        'console=tty0 console=ttyS0,115200'
      To
        'console=ttyS0,115200 console=tty0'
~~~~


### Kernel panic during the deployment

     This may be caused by the 'console' configuration in the kernel parameters which set by xCAT. You could try one of the following tips as a work around:
    1\. Unset all the serial console related attributes:

~~~~
      chdef <node> serialport= serialspeed= serialflow=
~~~~


    2\. Try the another serial port:

~~~~
      chdef <node> serialport=<another serial port>
~~~~


Note: You have to run the 'nodeset' after the above setting.

### Debug grub2 problems

grub2 has a debug option which could be specified in the /tftpboot/boot/grub2/grub.cfg-<mac>, here is an example, the set debug=all enables the grubs debug.

~~~~
[root@c910f02c01p02 grub2]# cat /tftpboot/boot/grub2/grub.cfg-01-e6-d4-dd-f8-1d-03
#install rhels7.1-ppc64-compute
set debug=all
set timeout=5
set default="xCAT OS Deployment"
menuentry "xCAT OS Deployment" {
    insmod http
    insmod tftp
    set root=tftp,10.2.1.2
    echo Loading Install kernel ...
    linux /xcat/osimage/rhels7.1-ppc64-install-compute/vmlinuz quiet inst.repo=http://10.2.1.2:80/install/rhels7.1/ppc64 inst.ks=http://10.2.1.2:80/install/autoinst/c910f02c01p03 BOOTIF=e6:d4:dd:f8:1d:03  ip=dhcp 
    echo Loading initial ramdisk ...
    initrd /xcat/osimage/rhels7.1-ppc64-install-compute/initrd.img
}
[root@c910f02c01p02 grub2]# 
~~~~
### ssh keys setup on Node do not work

Check to make sure SELinux is disable on the node.

## How to debug a virtual machine management issue

### kvm problems

Using the xnba instead of pxe as the netboot method.

Check the xCAT DB

Following xCAT tables are have definition information for virtual machine:

   Table vm: Includes most of the information of a virtual machine
   Table vmmater: If you are trying to do the clonevm, the vmmaster table needs to be used.
   Table kvm_nodedata. After the mkvm of a vm. You can find the kvm definition has been added in the kvm_nodedata.xml. Show it by 'nodels &lt;vm&gt; kvm_nodedata.xml'.

   Check the location and permission of storage (nfs://)

The nfs server can be set on MN or KVM hypervisor, pay attention to the permission of the storage path.

   Set the model of storage and nic



   Model of storage: default is 'dh'. The others valid value: scsi, virtio
   Model of nic: default is e1000. The others valid value: rtl8139, virtio

   Use the vnc to display the vm console

    The vnc port is opened when the vm is runing. You can open the vnc of vm by following approaches:

~~~~
      ssh -X to the kvm hypervisor,
      vnc login to the kvm hypervisor,
      Run 'vncviewer localhost:&lt;vnc port&gt;' to open the vnc to the vm.
    Note: the &lt;vnc port&gt; can be found from the 'lsdef vm' after the 'mkvm' run for the node.
~~~~

Use the virsh to debug



~~~~
   virsh list - display the running vm
   virsh dumpxml - dump the definition of vm
   virsh start <dom> \- start the vm
   virsh destroy <dom> \- destroy a vm
~~~~

Check the network process



Look into the definition of bridge, as default it is br0.
The configuration file /etc/sysconfig/network-scripts/ifcfg-br0 should includes following lines:

~~~~
           DEVICE=br0
           TYPE=Bridge
           ONBOOT=yes
           BOOTPROTO=dhcp
           PEERDNS=yes
           DELAY=0
~~~~




Look into the definition of nic which configured for bridge.
The configuration file /etc/sysconfig/network-scripts/ifcfg-eth0 (Use eth0 as example) should includes following lines:

~~~~
           DEVICE=eth0
           TYPE=Ethernet
           ONBOOT=yes
           BRIDGE=br0
           PROMISC=yes
           HWADDR=e4:1f:13:65:31:e8
~~~~




Display the configuration of bridge

~~~~
        # brctl  show
        bridge name     bridge id               STP enabled     interfaces
        br0             8000.e41f136531e8       no              eth0
                                                                vnet0
                                                                vnet1
                                                                vnet2

~~~~



Reset the network if you have changes for the network on kvm hypervisor

~~~~
       service network reset
~~~~


####Change the resource of a vm

Following resources can be changed for a vm: cpu, memory, disk, nic.

     Note: Certain changes need to reboot the vm or rmvm/mkvm again.

Look into the qemu log when start a vm

From the log, you can figure out which command is running when creating the virtual machine.

~~~~
        /var/log/libvirt/qemu/<vm>.log
~~~~


#### Reboot the libvirtd to handle the unknown problems

     Note: Reboot the libvirtd will cause all the running vm to be stopped.

## How to debug the discovery

### Discovery for system x

The discovery of the node can be used to map the node definition of xCAT to the physical node. After discovery the following attributes will be set for the node definition: bmcpassword,mac,mtm,serial.

After the discovery, if the mac was not filled automatically, check the switch and switches tables has been configured correctly

Hardware discovery for the System x machines need the physical network connection (The mapping of the node and the port of switch that the node directly connected to) information to perform.

Show the switch related attributes for the node. If not set, try to set it correctly.

~~~~
    lsdef <node> -i switch,switchport
~~~~


Add the correct access entry for the switch in the 'switches' table. The attributes snmpversion,username,password should be set at least.

Check snmp has been enabled for the switches

Run following command try to get the 'description information of this switch'. If failed, you need to check the configuration of the switch.

~~~~
    snmpget -v 3 -a sha -u <username> -A <password> -X <password> -l authNoPriv <host of switch> .1.3.6.1.2.1.1.1.0
~~~~


Check the dynamic range has been added into the dhcpd.conf for the correct network interface. If not:

~~~~
    chdef -t network <networkname> dynamicrange=startip-endip
    makedhcp -n
    service dhcpd restart
~~~~


Check the fixed IP configured in the configuration file dhcpd.leases has been removed

If not removed, run:

~~~~
    makedhcp -d <node>
    service dhcpd restart
    And check again!
~~~~


Check the nbk.&lt;arch&gt; and nbfs.&lt;arch&gt; have been created in the /tftpboot/xcat

###How to perform the discovery

Monitoring the status of the discovery from syslog: tail -f /var/log/messages

Walk over to systems, hit power buttons, then monitoring the following messages from /var/log/messages:

The discovery has been triggered for the node:    xcatd: Processing discovery request from &lt;dynamic IP of the node&gt;
The discovery for the node has succeeded:    &lt;node&gt; has been discovered


After the discovery

Check the attributes have been updated for the node. Especially the mac.mac.

Check the command 'rpower &lt;node&gt; stat' can get the correct status output.

Then run the makedhcp/nodeset/rpower to boot the node.

## How to boot up a node with debug kernel for Linux

1) Add the debug kernel to the pkglist:

~~~~
     kernel-debug
     kernel-debug-debuginfo
     kernel-debug-devel
     kernel-debuginfo
     kernel-debuginfo-common
     kernel-devel
~~~~


2) Run genimage

2.1) For xCAT 2.8.3 and later

~~~~
     genimage <imagename> -k 2.6.32-220.7.6.p7ih.el6.ppc64.debug -g 2.6.32-220.7.6.p7ih.el6.ppc64
~~~~


2.2) Before xCAT 2.8.3

~~~~
     genimage <imagename>
~~~~


3) Run packimage for stateless or liteiamge for statelite image

~~~~
     packimage <imagename>
~~~~


or

~~~~
     liteimage <imagename>
~~~~


4) Run geninitrd and specify the debug kernel

Note: Ignore this step for xCAT 2.8.3 and later.

For example:

~~~~
     /opt/xcat/share/xcat/netboot/rh/geninitrd -i hf0 -n hf_if -o rhels6 -k 2.6.32-220.7.6.p7ih.el6.ppc64.debug -p compute
~~~~


5) Run nodeset to assign the image to the node

~~~~
     nodeset <noderange> osimage=<imagename>
~~~~


6) Deploy the node

## Enable KDB for AIX Power 775 nodes

This procedure has been used to enable KDB for AIX Power 775 lpars:

~~~~
   xcatchroot -i <osimage> "bosdebug -D"
   nim -o check <osimage>
~~~~

     This does the equivalent of "bosboot -a" with a reboot, which is what is normally required to make it take affect on a diskfull node

~~~~
   xcatchroot -i <osimage> "bosdebug -L"
~~~~

    Should show:

~~~~
      Memory debugger           off
      Memory sizes              0
      Network memory sizes      0
      Kernel debugger           on
~~~~

