<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Overview and Background](#overview-and-background)
- [What is osimage?](#what-is-osimage)
- [Why do I want to convert to osimage based configuration?](#why-do-i-want-to-convert-to-osimage-based-configuration)
- [Procedure of using osimage](#procedure-of-using-osimage)
- [Convert to osimage based configuration](#convert-to-osimage-based-configuration)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

**Note: this documentation only works for xCAT 2.8 and earlier, for newer xCAT versions, see [Convert_Non-osimage_Based_System_To_Osimage_Based_System]**

![](https://sourceforge.net/p/xcat/wiki/XCAT_Documentation/attachment/Official-xcat-doc.png)


## Overview and Background

This documentation illustrates how to convert a Linux non-osimage based system to an osimage based system. This doc only works for xCAT on Linux, xCAT on AIX system uses a different mechanism and is not covered by this doc. 

In the old xCAT releases, the Linux os provisioning configuration was determined by the node attributes "os", "arch", "profile" and "provmethod"; xCAT code will internally use these four attributes to setup the os provisioning configuration, for example, if the node attributes are set to: 
    
     os=rhels6.3
     arch=x86_64
     profile=compute
     provmethod=install
    

The os provisioning configuration for this node will look like: 
    
     kickstart template(template): /opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.tmpl or /install/custom/install/rh/compute.rhels6.x86_64.tmpl
     os repository(pkgdir): /install/rhels6.3/x86_64
     package list file(pkglist): /opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.pkglist or /install/custom/install/rh/compute.rhels6.x86_64.pkglist
     otherpkgs directory(otherpkgdir) /install/post/otherpkgs/rhels6.3/x86_64/
     otherpkgs list(otherpkgslist): /install/custom/install/rh/compute.rhels6.x86_64.otherpkgs.pkglist
     synclist file(synclists): /install/custom/install/rh/compute.rhels6.x86_64.pkglist
    

We can see that the path of configuration files are determined implicitly based on the node attributes "os", "arch", "profile" and "provmethod"; further more, the search for some of the configuration files like template and pkglist uses the longest prefix matching algorithm, it sometimes will confuse the administrators and users. Another problem is that you could not customize too much for the os provisioning configuration. 

In some recent xCAT release(actually not that recent, it was first announced with xCAT 2.3 in year 2009, and a lot of enhancements were done later.), a new concept "osimage" was introduced to address all of the problems with the old way of using "os", "arch", "profile" and "provmethod". Till now, the osimage feature has been tested thoroughly and has been used widely by different teams for a while. 

## What is osimage?

The osimage is a type of xCAT object definition that can be managed through mkdef,lsdef,chdef,rmdef commands, the osimage object contains the configuration information for the os provisioning, including the information in the example mentioned above, after the osimage is created and customized correctly, it could be associated with any node that uses the os provisioning configuration defined in the osimage. xCAT will create some osimage definitions by default when you run copycds. The osimage information is stored in xCAT tables osimage and linuximage, you could see more details about osimage in the [osimage](http://xcat.sourceforge.net/man5/osimage.5.html) and [linuximage](http://xcat.sourceforge.net/man5/linuximage.5.html) manpages. 

## Why do I want to convert to osimage based configuration?

**xCAT team strongly recommends to use the osimage configuration instead of the old way of using "os", "arch", "profile" and "provmethod"**, the osimage has several advantages comparing with the old way: 

1\. It is cleaner, all the configuration in the osimage are set explicitly, you do not need to calculate or remeber the location of the configuration files. You will not run into problems like that the os provisioning actually uses a different configuration file than you expected because of the implicit way of calculating the path and configuration files. 

2\. It is easier, you could use *def commands to manage the osimage easily, the commands syntax is exactly the same with the nodes management, the only thing you need to is to add a flag "-t osimage" to let *def commands know that you are managing osimage objects. 

3\. It is more flexible, the osimage provides many configurable options that can be used to customize the os provisioning configuration. If you do not want to customize too much, the default osimage objects created by copycds should be able to satisfy your requirements. 

## Procedure of using osimage

The use of osimage does not change the general Linux os provisioning procedure, the only difference is that you need to specify the osimage when you run nodeset command. The procedure of using osimage has been covered by different xCAT docs like [XCAT iDataPlex Cluster Quick Start](https://sourceforge.net/apps/mediawiki/xcat/index.php?title=XCAT_iDataPlex_Cluster_Quick_Start) and [xCAT pLinux Clusters](https://sourceforge.net/apps/mediawiki/xcat/index.php?title=XCAT_pLinux_Clusters), here is only a summary of the procedure of using osimage: 

1\. Create the osimage 

xCAT command copycds will create some osimage definitions for the operating system being copied, these default osimage definitions could be used for different os provisioning scenarios. Here is an example list of osimage definitions created by copycds for x86_64 RHEL 6.3: 
    
    [root@xcatmn ~]# lsdef -t osimage
    rhels6.3-x86_64-install-compute  (osimage)
    rhels6.3-x86_64-install-compute_ad  (osimage)
    rhels6.3-x86_64-install-hpc  (osimage)
    rhels6.3-x86_64-install-iscsi  (osimage)
    rhels6.3-x86_64-install-kvm  (osimage)
    rhels6.3-x86_64-install-service  (osimage)
    rhels6.3-x86_64-install-storage  (osimage)
    rhels6.3-x86_64-install-xen  (osimage)
    rhels6.3-x86_64-netboot-compute  (osimage)
    rhels6.3-x86_64-netboot-kvm  (osimage)
    rhels6.3-x86_64-netboot-nfsroot  (osimage)
    rhels6.3-x86_64-netboot-service  (osimage)
    rhels6.3-x86_64-netboot-xen  (osimage)
    rhels6.3-x86_64-statelite-compute  (osimage)
    rhels6.3-x86_64-statelite-kvm  (osimage)
    rhels6.3-x86_64-statelite-nfsroot  (osimage)
    rhels6.3-x86_64-statelite-service  (osimage)
    rhels6.3-x86_64-statelite-xen  (osimage)
    [root@xcatmn ~]#
    

If you want to create new osimage definitions for whatever reason, you could use mkdef to create osimage definitions, here is an example: 
    
     mkdef -t osimage myosimage imagetype=linux osarch=x86_64 \
       osname=Linux osvers=rhels6.3 otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64 \
       pkgdir=/install/rhels6.3/x86_64 pkglist=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.pkglist \
       profile=compute provmethod=install template=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.tmpl
    

2\. Customize the osimage definitions 

You could use chdef command to change the osimage definitions, there are a lot of configurable options for osimage definitions, you could use 
    
     lsdef -t osimage -h
    

to list the attributes of osimage definitions. To change an attribute, run the command like: 
    
     chdef -t osimage myosimage pkglist=/install/custom/install/rh/myosimage.pkglist
    

3\. Create diskless images 
    
     genimage &lt;osimage_name&gt;
     packimg &lt;osimage_name&gt;
     or
     liteimg &lt;osimage_name&gt;
    

4\. Associate the osimage with nodes 

nodeset command now accepts the osimage as a parameter, you could run nodeset command to associate the osimage with nodes. 
    
     nodeset &lt;noderange&gt; osimage=&lt;osimage_name&gt;
    

The nodeset command will set the node attribute provmethod to be the osimage name, if the nodes' provmethod has already been set to be an osimage, or you need to run nodeset against nodes with different osimages in one invocation, you could run nodeset command with the keyword osimage(this is a feature available only in xCAT 2.8 and later releases): 
    
     nodeset &lt;noderange&gt; osimage
    

## Convert to osimage based configuration

The core part of converting to osimage based configuration is to translate the current os provisioning configuration into the osimage definitions. 

1\. Categorize the current os provisioning configurations 

The existing os provisioning configuration is determined by the node attributes "os", "arch", "profile" and "provmethod", the following commands could be used to categorize the os provisioning configuration: 
    
     lsdef -t node -i os,arch,profile,provmethod -c | xdshbak -c
     
     or
     
     tabdump nodetype | awk -F',' '{print $2,$3,$4,$5,$1}' | sort
    

Here is an example output of the _lsdef -t node -i os,arch,profile,provmethod -c | xdshbak -c_: 
    
    [root@xcatmn ~]# lsdef -t node -i os,arch,profile,provmethod -c | xdshbak -c
    
    HOSTS:
    -------------------------------------------------------------------------
    node01,node02,node03,node04,node05,node06,node07,node08,node09,node10
    -------------------------------------------------------------------------------
    arch=x86_64
    os=rhels6.3
    profile=compute
    provmethod=install
    
    HOSTS:
    -------------------------------------------------------------------------
    node11,node12,node13,node14,node15,node16,node17,node18,node19,node20
    -------------------------------------------------------------------------------
    arch=x86_64
    os=rhels6.3
    profile=compute
    provmethod=netboot
    
    HOSTS:
    -------------------------------------------------------------------------
    node21,node22,node23,node24,node25,node26,node27,node28,node29,node30
    -------------------------------------------------------------------------------
    arch=x86_64
    os=rhels6.3
    profile=compute
    provmethod=statelite
    [root@xcatmn ~]#
    
    

According to the output, there are three os provisioning configuration categories, each category has 10 nodes. 

2\. (Optional) Create node groups for each os provisioning configuration category 

To simplify the subsequent steps, it is a good idea to add these nodes in each os provisioning configuration category to different node groups. 
    
     chdef node01-node10 -p groups=osimage1
     chdef node11-node20 -p groups=osimage2
     chdef node21-node30 -p groups=osimage3
    

3\. List the os provisioning configuration for each category 

Select one node inside each os provisioning configuration category, run the following command to list the details of the os provisioning configuration: 
    
     [root@xcatmn ~]# lsdef node01 --osimage
    Object name: node01
    ... ... 
       status=booted
       statustime=12-18-2012 18:47:48
       provmethod=install
       profile=compute
       template=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.tmpl
       pkglist=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.pkglist
       osvers=rhels6.3
       osarch=x86_64
       osname=Linux
       imagetype=linux
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64
       pkgdir=/install/rhels6.3/x86_64
     [root@xcatmn ~]#
    
    
     [root@xcatmn ~]# lsdef node11 --osimage
    Object name: node11
    ... ... 
       status=booted
       statustime=12-18-2012 16:37:28
       postinstall=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.postinstall
       rootimgdir=/install/netboot/rhels6.3/x86_64/compute
       provmethod=netboot
       profile=compute
       pkglist=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.pkglist
       osvers=rhels6.3
       osarch=x86_64
       osname=Linux
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64
       imagetype=linux
       extlist=/opt/xcat/share/xcat/netboot/rh/compute.exlist
       pkgdir=/install/rhels6.3/x86_64
     [root@xcatmn ~]#
    
    
     [root@xcatmn ~]# lsdef node21 --osimage
    Object name: node21
    ... ... 
       status=booted
       statustime=12-18-2012 15:12:35
       postinstall=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.postinstall
       rootimgdir=/install/netboot/rhels6.3/x86_64/compute
       provmethod=statelite
       profile=compute
       pkglist=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.pkglist
       osvers=rhels6.3
       osarch=x86_64
       osname=Linux
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64
       imagetype=linux
       extlist=/opt/xcat/share/xcat/netboot/rh/compute.exlist
       pkgdir=/install/rhels6.3/x86_64
     [root@xcatmn ~]#
    

The attributes after the "statustime" attribute store the information for os provisioning configuration. 

4\. Create osimage for each os provisioning configuration 

Before creating new osimages manually, you might want to check if the default osimage definitions created by xCAT could match your requirements. 
    
     [root@xcatmn ~]# lsdef -t osimage
     rhels6.3-x86_64-install-compute  (osimage)
     rhels6.3-x86_64-install-compute_ad  (osimage)
     rhels6.3-x86_64-install-hpc  (osimage)
     rhels6.3-x86_64-install-iscsi  (osimage)
     rhels6.3-x86_64-install-kvm  (osimage)
     rhels6.3-x86_64-install-service  (osimage)
     rhels6.3-x86_64-install-storage  (osimage)
     rhels6.3-x86_64-install-xen  (osimage)
     rhels6.3-x86_64-netboot-compute  (osimage)
     rhels6.3-x86_64-netboot-kvm  (osimage)
     rhels6.3-x86_64-netboot-nfsroot  (osimage)
     rhels6.3-x86_64-netboot-service  (osimage)
     rhels6.3-x86_64-netboot-xen  (osimage)
     rhels6.3-x86_64-statelite-compute  (osimage)
     rhels6.3-x86_64-statelite-kvm  (osimage)
     rhels6.3-x86_64-statelite-nfsroot  (osimage)
     rhels6.3-x86_64-statelite-service  (osimage)
     rhels6.3-x86_64-statelite-xen  (osimage)
     [root@xcatmn ~]# lsdef -t osimage rhels6.3-x86_64-install-compute
     Object name: rhels6.3-x86_64-install-compute
       imagetype=linux
       osarch=x86_64
       osdistroname=rhels6.3-x86_64
       osname=Linux
       osvers=rhels6.3
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64
       pkgdir=/install/rhels6.3/x86_64
       pkglist=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.pkglist
       profile=compute
       provmethod=install
       template=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.tmpl
     [root@xcatmn ~]# 
    

  
If the default osimage definitions created by xCAT could not match your requirements, or the default osimage definitions are not created on your xCAT management node at all, you need to create one osimage definition for each os provisioning configuration, as mentioned in the step #1. 
    
     mkdef -t osimage osimage1  template=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.tmpl \
       pkglist=/opt/xcat/share/xcat/install/rh/compute.rhels6.x86_64.pkglist \
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64 \
       pkgdir=/install/rhels6.3/x86_64 \
       osname=Linux \
       imagetype=linux \
       osarch=x86_64 \
       osvers=rhels6.3 \
       profile=compute \
       provmethod=install
    

  

    
     mkdef -t osimage osimage2  rootimgdir=/install/netboot/rhels6.3/x86_64/compute \
       postinstall=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.postinstall \
       permission=755 \
       pkglist=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.pkglist \
       exlist=/opt/xcat/share/xcat/netboot/rh/compute.exlist \
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64 \
       pkgdir=/install/rhels6.3/x86_64 \
       imagetype=linux \
       osname=Linux \
       osarch=x86_64 \
       osvers=rhels6.3 \
       profile=compute \
       provmethod=netboot
    
    
     mkdef -t osimage osimage3  rootimgdir=/install/netboot/rhels6.3/x86_64/compute \
       postinstall=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.postinstall \
       permission=755 \
       pkglist=/opt/xcat/share/xcat/netboot/rh/compute.rhels6.x86_64.pkglist \
       exlist=/opt/xcat/share/xcat/netboot/rh/compute.exlist \
       otherpkgdir=/install/post/otherpkgs/rhels6.3/x86_64 \
       pkgdir=/install/rhels6.3/x86_64 \
       imagetype=linux \
       osname=Linux \
       osarch=x86_64 \
       osvers=rhels6.3 \
       profile=compute \
       provmethod=statelite
    

5\. Create diskless images 

For stateless and statelite osimages, needs to run genimage and packimg/liteimg to create the diskless image. 
    
     genimage osimage1
     genimage osimage2
     genimage osimage3
    

  

    
     packimg osimage1
     packimg osimage2
     packimg osimage3
    
    
     or
     liteimg osimage1
     liteimg osimage2
     liteimg osimage3
    

6\. Associate the osimage with the nodes 

You could run: 
    
     nodeset node01-node10 osimage=osimage1
     nodeset node11-node20 osimage=osimage2
     nodeset node21-node30 osimage=osimage3
     
     or if you are running xCAT 2.8 or later release:
     
     chdef node01-node10 provmethod=osimage1
     chdef node11-node20 provmethod=osimage2
     chdef node21-node30 provmethod=osimage3
     nodeset node01-node30 osimage
    

**Note:** for diskful nodes, you might not want to run nodeset to avoid re-provisioning when the diskful nodes are rebooted. If this is the case, just run chdef to set the provmethod to be the osimage is enough. 