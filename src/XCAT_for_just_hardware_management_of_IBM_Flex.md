![](https://sourceforge.net/p/xcat/wiki/XCAT_Documentation/attachment/Official-xcat-doc.png)


In some environments, the hardware setup and management functions are desired without the OS deployment, DNS, or other capabilites. This document describes how to install xCAT and describe your desired configuration and have xCAT push it out. 

First, to install xCAT, the same repositories should be added to a system as would be used for a typical install as covered in other documents. However, rather than installing the xCAT package, just xCAT-server and its prerequsites are installed. This can be done a number of ways, but for reference, the commands done for creating this document: 

~~~~
    
 cd /etc/yum.repos.d/
 wget http://sourceforge.net/projects/xcat/files/yum/stable/core-snap/xCAT-core.repo
 wget http://sourceforge.net/projects/xcat/files/yum/xcat-dep/rh6/x86_64/xCAT-dep.repo/download
 yum install xCAT-server perl-DBD-SQLite
 . /etc/profile.d/xcat.sh 
~~~~    

In this example, IPv6 will be used. 

~~~~    
 yum install perl-Socket6 perl-IO-Socket-INET6
~~~~    

The IPv6 support may be skipped if the implementation will not be using IPv6, and IPv4 addresses are allocated for each and every IMM. If the IMMs must not have an IPv4 address, then IPv6 will be used behind the scenes to address them without administrator allocating them. This scenario is detailed later in this document. 

Next, global configuration parameters are specified.. For this example, groups are declared with some parameters that make sense for an IBM Flex environment. The group representing the ITEs will be named 'flex' and the group representing the CMMs will be named 'cmm'. Because the xCAT metapackage was not installed, some configuration is going to be done manually: 
 
~~~~   
 chtab key=xcatdport site.value=3001
 /opt/xcat/share/xcat/scripts/setup-xcat-ca.sh "xCAT"
 /opt/xcat/share/xcat/scripts/setup-server-cert.sh `hostname`
 /opt/xcat/share/xcat/scripts/setup-local-client.sh root
 chtab name=root policy.rule=allow policy.priority=1 policy.commands=*
 service xcatd start
 makenetworks
~~~~    

SSH user key is required for root on the management server (accept default suggested parameters): 
  
~~~~  
     # ssh-keygen -t rsa   
~~~~    

Next, we want members of flex to use ipmi for management: 
   
~~~~ 
     # nodegrpch flex nodehm.mgt=ipmi 
~~~~    

And for cmm, we will use blade for management: 

~~~~    
     # nodegrpch cmm nodehm.mgt=blade
~~~~    

In general, xCAT configuration can be done in more complex ways on a group. This example uses a naming scheme consisting of n1,n2,etc. where 1-14 are connected to cmm1, 15-28 to cmm2, etc. This means that cmm1 is the chosen resolvable name to whatever address applies to the environment. Vary as appropriate. Configuration may alternatively be specified in a more straightforward, node by node way. mp.mpa is the resolvable name of the CMM, mp.id is the slot number: 
  
~~~~  
nodegrpch flex mp.mpa='|\D+(\d+)\z|cmm((($1-1)/14)+1)|' mp.id='|\D(\d+)\z|((($1-1)%14)+1)|'
~~~~    

There are xCAT facilities for manipulating name resolution and easing this activity for at-scale configurations, but this is considered out of scope for this document. 

For the cmms, this is straightforward, they are all set to be managed by themselves, slot 0: 
    
nodegrpch cmm mp.mpa='|||' mp.id=0
    

Systems management passwords must be specified. In this example, Passw0rd is chosen for illustrative purposes. 
    
~~~~
 chtab key=blade passwd.username=USERID passwd.password=Passw0rd
 chtab key=ipmi passwd.username=USERID passwd.password=Passw0rd
~~~~    

Note that this is the password that you ultimately wish to use. It might not match the current password of the configuration. 

The CMMs must then be declared to xCAT: 
  
~~~~  
     # nodeadd cmm1-cmm2 groups=cmm 
~~~~    

From here there are two likely scenarios. If manual configuration is done for the CMMs, rscan may be used to show what ITEs are available. This documentation is going to continue on the alternate path, that the CMMs still need configuration. 

This example is going to locate the CMMs according to where they are plugged into a switch. In this case, the switch is configured to allow SNMPv1 community string 'public' to read. For other authentication scenarios (e.g. SNMPv3 authentication required), please see 'man switches' for further information. 

If desired, a scheme resembling the mp.mpa/mp.id can be configured for CMM map to switches. However since this example contains only have two chassis, it suffices to specify each one individually: 
   
~~~~ 
 nodech cmm1 switch.switch=r15e1 switch.port=47
 nodech cmm2 switch.switch=r15e1 switch.port=48
~~~~    

It is possible to either defer defining nodes until later after the CMMs are discovered, or add them now. This example shows us adding them now. 
  
~~~~  
 nodeadd n1-n26 groups=flex
~~~~    

It is possible to defer this step and add later if, for example, there is a need to inventory the chassis to know how many systems are present before proceeding. Here it is known that there are 26 so it is declared ahead of time.. 

There are two general options for how one may want to deal with the IMM addressing. The addresses may be pushed down to discovered IMMs using ipmi.bmc (ipv4 or ipv6) or take no action to let xCAT auto-fill the addresses it finds (IPv6 support required). Neither step requires the user to pre-configure the addresses on the IMMs, xCAT will handle it using the same command set. This example will proceed for now with the auto-detect support. 

These CMMs already have been set with the password 'TestD3mo'. We specify that at the command line. After the command runs, TestD3mo will no longer work to log into the CMMs. 
    
~~~~
     # XCAT_CURRENTPASS=TestD3mo lsslp --flexdiscover                                                                 
     cmm1: Found service:management-hardware.IBM:chassis-management-module at address fe80::5ef3:fcff:fe25:e533%3
     Configuration of n11[fe80::5ef3:fcff:fe6e:14dc%3] commencing, configuration may take a few minutes to take effect
     Configuration of n8[fe80::5ef3:fcff:fe6e:1578%3] commencing, configuration may take a few minutes to take effect
     Configuration of n13[fe80::5ef3:fcff:fe6e:1570%3] commencing, configuration may take a few minutes to take effect
     Configuration of n9[fe80::5ef3:fcff:fe6e:1510%3] commencing, configuration may take a few minutes to take effect
     Configuration of n5[fe80::5ef3:fcff:fe6e:13e0%3] commencing, configuration may take a few minutes to take effect
     Configuration of n2[fe80::5ef3:fcff:fe6e:143c%3] commencing, configuration may take a few minutes to take effect
     Configuration of n14[fe80::5ef3:fcff:fe6e:1538%3] commencing, configuration may take a few minutes to take effect
     Configuration of n6[fe80::5ef3:fcff:fe6e:145c%3] commencing, configuration may take a few minutes to take effect
     Configuration of n10[fe80::5ef3:fcff:fe6e:13f8%3] commencing, configuration may take a few minutes to take effect
     Configuration of n12[fe80::5ef3:fcff:fe6e:1500%3] commencing, configuration may take a few minutes to take effect
     Configuration of n4[fe80::5ef3:fcff:fe6e:1548%3] commencing, configuration may take a few minutes to take effect
     Configuration of n3[fe80::5ef3:fcff:fe6e:14cc%3] commencing, configuration may take a few minutes to take effect
     Configuration of n1[fe80::5ef3:fcff:fe6e:1424%3] commencing, configuration may take a few minutes to take effect
     Configuration of n7[fe80::5ef3:fcff:fe6e:148c%3] commencing, configuration may take a few minutes to take effect
     cmm1: Configuration complete, configuration may take a few minutes to take effect
     cmm2: Found service:management-hardware.IBM:chassis-management-module at address fe80::5ef3:fcff:fe25:e4b3%3
     Configuration of n20[fe80::5ef3:fcff:fe6e:144d%3] commencing, configuration may take a few minutes to take effect
     Configuration of n17[fe80::5ef3:fcff:fe6e:1429%3] commencing, configuration may take a few minutes to take effect
     Configuration of n15[fe80::5ef3:fcff:fe6e:1585%3] commencing, configuration may take a few minutes to take effect
     Configuration of n24[fe80::5ef3:fcff:fe6e:13e9%3] commencing, configuration may take a few minutes to take effect
     Configuration of n26[fe80::5ef3:fcff:fe6e:159d%3] commencing, configuration may take a few minutes to take effect
     Configuration of n16[fe80::5ef3:fcff:fe6e:14f1%3] commencing, configuration may take a few minutes to take effect
     Configuration of n21[fe80::5ef3:fcff:fe6e:157d%3] commencing, configuration may take a few minutes to take effect
     Configuration of n25[fe80::5ef3:fcff:fe6e:14c1%3] commencing, configuration may take a few minutes to take effect
     Configuration of n19[fe80::5ef3:fcff:fe6e:14b5%3] commencing, configuration may take a few minutes to take effect
     Configuration of n18[fe80::5ef3:fcff:fe6e:1531%3] commencing, configuration may take a few minutes to take effect
     Configuration of n23[fe80::5ef3:fcff:fe6e:1595%3] commencing, configuration may take a few minutes to take effect
     Configuration of n22[fe80::5ef3:fcff:fe6e:1479%3] commencing, configuration may take a few minutes to take effect
     cmm2: Configuration complete, configuration may take a few minutes to take effect
  
~~~~  

Note: the command slpdiscover has been replaced by the command lsslp --flexdiscover in xCAT 2.8. If you are using lower version ( xCAT 2.7.x ), please use command 'XCAT_CURRENTPASS=TestD3mo slpdiscover'. 

At this point, full access to hardware management is available, including power on/off/reset: 
   
~~~~ 
     # rpower flex stat|xcoll
     ====================================
     flex
     ====================================
     off
~~~~    

Inventory: 
  
~~~~  
     # rinv n1-n2 vpd
     n1: System Description: IBM Flex System x240+10Gb Fabric
     n1: System Model/MTM: 8737AC1
     n1: System Serial Number: 23XXH49
     n1: Chassis Serial Number: 23XXH49
     n1: Device ID: 32
     n1: Manufacturer ID: IBM (20301)
     n1: BMC Firmware: 1.60 (1AOO31P 2012/08/07 22:40:32)
     n1: Product ID: 321
     n2: System Description: IBM Flex System x240+10Gb Fabric
     n2: System Model/MTM: 8737AC1
     n2: System Serial Number: 23XXH30
     n2: Chassis Serial Number: 23XXH30
     n2: Device ID: 32
     n2: Manufacturer ID: IBM (20301)
     n2: BMC Firmware: 1.34 (1AOO27Q 2012/05/04 22:00:54)
     n2: Product ID: 321
     # [root@xcat6 ~]# rinv n1-n2 mac
     n1: MAC Address 1: 5c:f3:fc:6e:47:08
     n1: MAC Address 2: 5c:f3:fc:6e:47:0c
     n2: MAC Address 1: 5c:f3:fc:6e:47:38
     n2: MAC Address 2: 5c:f3:fc:6e:47:3c
     n2: Mezz Exp 2 Board MAC Address 1: 00:00:c9:e5:29:66
     n2: Mezz Exp 2 Board MAC Address 2: 00:00:c9:e5:29:6a
     n2: Mezz Exp 2 Board MAC Address 3: 00:00:c9:e5:29:6e
     n2: Mezz Exp 2 Board MAC Address 4: 00:00:c9:e5:29:72
     [root@xcat6 ~]# rinv n1-n2 wwn
     n2: WWN 1: 10:00:5c:f3:fc:6e:47:39
     n2: WWN 2: 10:00:5c:f3:fc:6e:47:3d
     n1: WWN 1: 10:00:5c:f3:fc:6e:47:09
     n1: WWN 2: 10:00:5c:f3:fc:6e:47:0d
 
~~~~   

Identify light: 
  
~~~~ 
     # rbeacon flex on|xcoll
     ====================================
     flex
     ====================================
     on
~~~~     
    

Remote leds: 
  
~~~~  
     # rvitals flex leds|xcoll 
     ====================================
     flex
     ====================================
     No active error LEDs detected
~~~~    

  
Set one time PXE boot: 
 
~~~~   
     # rsetboot n1-n26 net|xcoll
     ====================================
     flex
     ====================================
     Network
~~~~    