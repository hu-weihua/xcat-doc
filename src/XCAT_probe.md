<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [External interface:](#external-interface)
- [Internal implementation:](#internal-implementation)
  - [xcatprobe noderange](#xcatprobe-noderange)
  - [xcatprobe -i osimage](#xcatprobe--i-osimage)
  - [xcatprobe -x](#xcatprobe--x)
  - [xcatprobe -s](#xcatprobe--s)
- [Performance consideration:](#performance-consideration)
  - [The format of the output:](#the-format-of-the-output)
- [Other Design Considerations](#other-design-considerations)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

{{:Design Warning}} 

For a long time, I noticed that xCAT was not easy to use for a new hand or even a person who has used it for a while. They might encounter issues that were caused by the MN configuration, installation of xCAT, node definition, services needed for provision (tftp,http,dns), image resource, network ... and mostly it was not easy for them to figure out what was the problem. This was a problem even for the xCAT FVT and DEV when they tried to use/test the xCAT functions that were not familiar for them, for example to manage a system x or blade node for one who was working for system p. Then I had an strong intention to implement a xCAT command to probe potential issues of xCAT, it also can be used to debug xCAT issues. 

A item for the xCAT probe was there for a long time (I remember from 2.6), but we always thought it was low priority and has been deferred for two releases. I think it's time to start some work for this item. I known our DEV resource is tight in 2012, so I'd like to split this item to several phases. I'll try to figure out a base version for internal use to collect comments and start the next phase when it's necessary. (The prototype for the first phase is almost ready, I can check it in soon) 

Command is named 'xcatproble'. It will be used to probe any xCAT related problems. The command request will be sent to xcatd so that the hierarchy can be supported. I referred to the doc 'Health Check Script Framework' that covers the checking with scripts. My opinion is the checking for the core functions of xCAT like&nbsp;:cfg of mn/sn, provisioning, osimage, cfg of network should be done through xCAT plugins (easy to code, better performance and easy to maintain), the functions for the checking of specific node like 'IB, HPC software' will be done by the scripts that easy for customer to customize (In next phase). 

  
The mini design for the xcatprobe, any comments are appreciated. 


## External interface:

xcatprobe noderange [-V] 
    
    Check the node definition; 
    Check the install/netboot resource; 
    Check the configuration files for deployment; 
    Check the node status after the deployment; (Next phase)
    Check the database configuration; 
    

Comments: LKV: This seems really difficult since there are so many allowable configurations. 

[wxp]: I think just check which DB is used and is the configuration of db correct. 

LKV: I really meant all of the above checks. What would be the rule for a correct node definition? Even checking a correct database configuration is hard. You could check /etc/xcat/cfgloc for which database is running and then run lsxcatd -d, to make sure that is the one running. 

xcatprobe -i osimage [-V] 
    
    Check the definition of osimage;
    Check the configuration files that were/will be used for genimage or installation;
    Check the kernel, initrd, rootimage that generated by genimage;
    

xcatprobe -x [-V] 
    
    Check the installation of xCAT for MN/SN. This can be verified by running lsxcatd -a on the MN and SN and compare
    Check the processes of xcatd on MN/SN. (lsxcat -a)
    

[wxp] Good point, I'll try to do it this way. 
    
    Check the installation of bootloader on MN/SN
    Check the nbroot (for disovery,bmcsetup) on MN/SN for x86 system
    

xcatprobe -s [-V] 
    
    Check the service syslog,dhcpd,tftpd,httpd on MN/SN
    

xcatprobe -n [-V] 
    
    Check the network configuration (Next phase)
    

  


## Internal implementation:

### xcatprobe noderange

Check the validation of attributes for nodes. Check the atts: os, arch, profile, netboot, netserver, installnic, mac, mgt, provmethod, bmc, hcp ... 

Check the osimage for the nodes. Refer to the 'xcatprobe -i'. 

Check the provision resource for the nodes. Check the dhcp cfg in the dhcp lease file; Check the bootloader configuration files for yaboot, pxe, xnba; Check the autoyast/anaconda cfg file; (Only check the existence of the files in first phase) 

Check the node status after the deployment; (Next phase) 

### xcatprobe -i osimage

Get the osimage of information os, arch, provmethod and profile from osimage name. xCAT supports two ways to specify the osimage: 1. with full name&nbsp;; 2. by the '-o -a -p' flag. During the check, we cannot figure out how the osimage was generated, so check the osimage with both format. 

For the full name osimage 

  * Get the configuration files from osimage and linuximage tables: template,pkglist,otherpkgs,postinstall,synclist 
  * Check the existence of these files 
  * Check the existence of initrd-stateless.gz,initrd-statelite.gz,kernel in the /install/netboot/os ... and /tftpboot/xcat/os ... 
  * Check the existence of image.gz in the /instal/netboot/os ... 

For the osimage specified by -a -p -o 

  * Search all the configuration files from the default path /install/custom or /opt/xcat/share/ ..., and list them out 
  * Check the existence of initrd-stateless.gz,initrd-statelite.gz,kernel in the /install/netboot/os ... and /tftpboot/xcat/os ... 
  * Check the existence of image.gz in the /instal/netboot/os ... 

### xcatprobe -x

  * Check the installation of xCAT packages: perl-xCAT, xCAT-client, xCAT-server, xCAT/xCATsn 
  * Check the installation of extending package: xCAT-IBMhpc, xcAT-UI, xCAT-test 
  * Check the xcatd processes: SSL listener, DB Access, UDP listener, install monitor 
  * Check the bootloader files in /tftpboot: yaboot, pxelinux.0, xnba.kpxe, xnba.efi 
  * Check the kernel and initrd of nbroot system for discovery/bmcsetup (system x) 

### xcatprobe -s

Check the service syslog,dhcpd,tftpd,httpd are running on MN/SN. 

## Performance consideration:

The running of the checking on the separated service nodes will be done parallelly; 

On MN/SN, the database access should be done in one access for all nodes. 

The output message for the node specific should be displayed in summarized format for all the nodes instead of one by one. The nodes which have the same error will be displayed together. 

### The format of the output:

I used the Term::ANSIColor module to display the output in color to identify the error/warning message. 

## Other Design Considerations

  * **Required reviewers**: 
  * **Required approvers**: Bruce Potter 
  * **Database schema changes**: N/A 
  * **Affect on other components**: N/A 
  * **External interface changes, documentation, and usability issues**: N/A 
  * **Packaging, installation, dependencies**: N/A 
  * **Portability and platforms (HW/SW) supported**: N/A 
  * **Performance and scaling considerations**: N/A 
  * **Migration and coexistence**: N/A 
  * **Serviceability**: N/A 
  * **Security**: N/A 
  * **NLS and accessibility**: N/A 
  * **Invention protection**: N/A 