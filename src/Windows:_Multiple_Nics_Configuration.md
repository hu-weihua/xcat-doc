<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Overview](#overview)
- [Interface](#interface)
  - [Set nics Attributes](#set-nics-attributes)
  - [Set Installnic](#set-installnic)
  - [Set Gateway Attribute](#set-gateway-attribute)
  - [Generate Configuration](#generate-configuration)
- [Example](#example)
- [Other Design Considerations](#other-design-considerations)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

{{:Design Warning}} 


## Overview

There's no feature in xCAT to support the configuration of network interfaces for Windows compute node. Currently, xCAT only uses the default setting of nics for them to get configuration from dhcp server. 

This design is used to describe how to use nics table to configure the network cards during Windows deployment. 

## Interface

The interface name in Windows is like 'Local Area Connection' (first interface), 'Local Area Connection 2' (second interface) and 'Local Area Connection x' (next interface). Since the ethernet name in Windows compute node includes spaces, both xCAT and PCM need to put effort to handle the ethernet name correctly. 

### Set nics Attributes

You need to specify the IP for the network interfaces in nics table like following. (xCAT will support to set it by chdef command.) 
    
    nics table
    #node,nicips,nichostnamesuffixes,nictypes,niccustomscripts,nicnetworks,nicaliases,comments,disable
    "x3550m4n03","Local Area Connection 3!192.168.13.250,Local Area Connection 2!192.168.12.250",,,,,,,
    
    
    networks table   
    #netname,net,mask,mgtifname,gateway,dhcpserver,tftpserver,nameservers,ntpservers,logservers,dynamicrange,staticrange,staticrangeincrement,nodehostname,ddnsdomain,vlanid,domain,comments,disable
    "192_168_13_0-255_255_255_0","192.168.13.0","255.255.255.0",,"192.168.13.254",,,,,,,,"1",,,,,,
    

PCM configures the bmc interface in the nics table with ethernet name 'bmc', since the 'bmc' interface should not configured in windows compute node, xCAT will ignore the ethernet name 'bmc' when creating the unattend.xml. 

In this example, the IP of interface 'Local Area Connection 2' will be set to '192.168.12.250'. And IP of interface 'Local Area Connection 3' will be set to '92.168.13.250'. Since no setting for 'Local Area Connection' (first interface) is specified, it will keep to use dhcp (this is the default setting in Windows node). 

**Note:** the network for the interfaces must be set correctly in networks table, otherwise the interface will be ignored if cannot find correct netmask for the ip from networks table. 

### Set Installnic

The installnic will only be set to static when the site.setinstallnic is set to '1' or 'yes', otherwise the installnic will keep to get IP from dhcp server even if it has been set in nics table. 

The node.installnic or node.primary is used specify the name of instlalnic. For Windows deployment, it must be specified. Otherwise xCAT will consider all the interfaces in nics table as non-installnic. 
    
    chdef -t site clustersite setinstallnic=1
    chdef &lt;node&gt;  installnic='Local Area Connection'
    

  


### Set Gateway Attribute

Only the gateway from the intallnic network will be set to default gateway for Windows compute node. The gateway which is set in other networks will be ignored. 

### Generate Configuration

Run nodeset command will make the setting take effect. 
    
    nodeset &lt;node&gt; osimage=win2k8r2-x86_64-install-enterprise
    

Then you can check the setting of component 'Microsoft-Windows-TCPIP in /install/autoinst/x3550m4n03.xml. Refer to http://technet.microsoft.com/en-us/library/ff716228.aspx 

## Example

To generated setting in /install/autoinst/x3550m4n03.xml base on the following setting in nics and networks tables. 
    
    nics table
    #node,nicips,nichostnamesuffixes,nictypes,niccustomscripts,nicnetworks,nicaliases,comments,disable
    "x3550m4n03","Local Area Connection 3!192.168.13.250,Local Area Connection 2!192.168.12.250",,,,,,,
    
    
    networks table   
    #netname,net,mask,mgtifname,gateway,dhcpserver,tftpserver,nameservers,ntpservers,logservers,dynamicrange,staticrange,staticrangeincrement,nodehostname,ddnsdomain,vlanid,domain,comments,disable
    "192_168_13_0-255_255_255_0","192.168.13.0","255.255.255.0",,"192.168.13.254",,,,,,,,"1",,,,,,
    

  
Configuration of interfaces: 
    
       &lt;component name="Microsoft-Windows-TCPIP" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"&gt;
          &lt;Interfaces&gt;
                 &lt;Interface wcm:action="add"&gt;
                        &lt;Ipv4Settings&gt;
                               &lt;DhcpEnabled&gt;false&lt;/DhcpEnabled&gt;
                           &lt;/Ipv4Settings&gt;
                        &lt;Ipv6Settings&gt;
                               &lt;DhcpEnabled&gt;false&lt;/DhcpEnabled&gt;
                           &lt;/Ipv6Settings&gt;
                        &lt;Identifier&gt;Local Area Connection 3&lt;/Identifier&gt;
                        &lt;UnicastIpAddresses&gt;
                               &lt;IpAddress wcm:action="add" wcm:keyValue="1"&gt;192.168.13.250/24&lt;/IpAddress&gt;
                           &lt;/UnicastIpAddresses&gt;
                        &lt;Routes&gt;
                               &lt;Route wcm:action="add"&gt;
                                      &lt;Identifier&gt;1&lt;/Identifier&gt;
                                      &lt;NextHopAddress&gt;192.168.13.254&lt;/NextHopAddress&gt;
                                      &lt;Prefix&gt;0/0&lt;/Prefix&gt;
                                  &lt;/Route&gt;
                           &lt;/Routes&gt;
                    &lt;/Interface&gt;
                 &lt;Interface wcm:action="add"&gt;
                        &lt;Ipv4Settings&gt;
                               &lt;DhcpEnabled&gt;false&lt;/DhcpEnabled&gt;
                           &lt;/Ipv4Settings&gt;
                        &lt;Ipv6Settings&gt;
                               &lt;DhcpEnabled&gt;false&lt;/DhcpEnabled&gt;
                           &lt;/Ipv6Settings&gt;
                        &lt;Identifier&gt;Local Area Connection 2&lt;/Identifier&gt;
                        &lt;UnicastIpAddresses&gt;
                               &lt;IpAddress wcm:action="add" wcm:keyValue="1"&gt;192.168.12.250/24&lt;/IpAddress&gt;
                           &lt;/UnicastIpAddresses&gt;
                    &lt;/Interface&gt;
             &lt;/Interfaces&gt;
      &lt;/component&gt;
    

  


## Other Design Considerations

  * **Required reviewers**: 
  * **Required approvers**: Bruce Potter, William, Jarrod 
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