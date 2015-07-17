<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Overview](#overview)
  - [Required Reviewers](#required-reviewers)
  - [Required Approvers](#required-approvers)
  - [copycds](#copycds)
  - [without copycds](#without-copycds)
  - [Management node pkglist, otherpkgs, tmpl](#management-node-pkglist-otherpkgs-tmpl)
- [Other Design Considerations](#other-design-considerations)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

{{:Design Warning}} 

  



## Overview

To allow kits deployment on the Management Node to the Management node an osimage must be defined for the Management Node in the database. 

### Required Reviewers

  * Norman Nott 
  * Linda Mellor 
  * Kerry Bosworth 

### Required Approvers

  * Bruce Potter 

### copycds

copycds will be modified to recognize that if the ISO matches what is on the Management node, it will also define a default osimage for the Management node in the database, if not already there. This osimage name will be well-defined, so the xCAT code can recognize it as the osimage for the management node. The osimage attributes will be filled in with the correct defaults for the management node. 

The name of the osimage for the MN should match what would be generated by PCM. (Open to discussion). 
    
     **rhels6.X-ppc64-stateful-mgmtnode**
    

Here is the PCM generated Management Node entry: 
    
    [root@x3650m2n01 ~]# lsdef __mgmtnode
    Object name: x3650m2n01
       arch=x86_64
       groups=__mgmtnode,__ImageProfile_rhels6.3-x86_64-stateful-mgmtnode
       ip=10.1.0.205
       nicips=eth0!10.1.0.205,eth1!9.114.34.205
       os=rhels6.3
       postbootscripts=syncfiles,ospkgs,otherpkgs
       postscripts=syslog,remoteshell,syncfiles
       profile=compute
       **provmethod=rhels6.3-x86_64-stateful-mgmtnode**
       status=booted
       statustime=26-02-2013 14:10:57
       updatestatus=synced
       updatestatustime=02-26-2013 14:14:02
    

  


If copycds creates a MN osimage definition for the currently running os/distro/version on the MN, and if the provmethod attr for __mgmtnode (group def) is not set, do the equiv of "chdef -t group -o __mgmtnode provmethod=&lt;MN_osimage&gt;

Covered by [[defect 3370](https://sourceforge.net/p/xcat/bugs/3370/)] 

### without copycds

If the customer wants to deploy kits on the Management node without running copycds, they will be responsible for manually creating the osimage for the Management Node. 

### Management node pkglist, otherpkgs, tmpl

After discussion, it was decided we would not ship new files for the management node, we will use the current service node files. If the admin needs to taylor one of these files for the Management Node, they can just change the default in the Management Node osimage. 

copycds, will create an image for the Management node, with the same default attributes filled in as if it was for a service node, but with a new unique name for the Management node. 

For example, the osimage would have the same attributes set by default as the following osimage, if the MN was rehels6.1 on x86_64. 
    
    rhels6.1-x86_64-install-service
    

  


## Other Design Considerations

  * **Required reviewers**: Bruce Potter 
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