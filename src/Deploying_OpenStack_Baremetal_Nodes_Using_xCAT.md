<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Introduction](#introduction)
- [Basic Idea](#basic-idea)
- [Implementation](#implementation)
  - [Baremetal node discovery](#baremetal-node-discovery)
  - [Baremetal node registration](#baremetal-node-registration)
  - [Image creation and registration](#image-creation-and-registration)
- [Packaging](#packaging)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

[Design_Warning](Design_Warning)

**This page is under construction.**


## Introduction

Baremetal node deployment in OpenStack is not very mature yet. The released [nova-baremetal implementation](https://wiki.openstack.org/wiki/Baremetal) is not good enough to be used in real business world in terms of performance. And it lacks of node discovery, hardware control, firmware updates, sensor readings and monitoring etc. [Ironic](https://wiki.openstack.org/wiki/Ironic) will replace nova-baremetal in the future for OpenStack, but it will take a while for Ironic to become mature and have all these features. In the meanwhile, some customers demand to have baremetal solution for their OpenStack clouds that is fully functioning. To close this gap, we'll introduce an interim solution, that is using xCAT. The idea is that user will run OpenStack commands to acquire baremetal nodes. Underneath, we'll have OpenStack call xCAT to do the plumbing. To the user, it is seamless. They do not have to know xCAT at all. 

This solution will gradually fade away once the Ironic is fully functioning. And there is no plan to contribute this implementation back into OpenStack. 

## Basic Idea

**1\. Baremetal node discovery**

Once the hardware arrives and networks are setup, the admin can use xCAT's node discovery methods to automatically discovery the node, setup the BMCs and save the mac address, cpu count, memory size and hard disk sizes into xCAT database. 

**2\. Baremetal node registration**

The admin can run a command (new) to register the xCAT-discovered nodes as OpenStack baremetal nodes. This command takes an xCAT node range as input and will run **nova baremetal-node-crate** to register the nodes into OpenStack. 

**3\. OpenStack image registration**

The admin will generate the images for the user using xCAT's image generation command "genimage". The admin then runs a command (new) to add xCAT images in to the OpenStack glance database. This command will take an xCAT image name or image group name as input and run **glance image-create** to register the image. The image will have the xCAT image name as its property. 

**4\. Baremetal node deployment**

The tenant will create his/her own subnet and run **nova boot** with image name to acquire baremetal nodes. A new OpenStack baremetal driver, which is a nova Compute hypervisor driver, will be used in this phase to replace the nova-baremetal driver. The new baremetal driver will get the xCAT image name from the OpenStack image name, it will also get the xCAT node name, the new ip address that is generated by neutron. It then calls xCAT commands to deploy the node. It will also update the OpenStack with node status. 

**5\. Console (tbd)**

**6\. Managing the baremetal nodes**

The admin can manage the baremetal nodes using xCAT commands. 

  * rinv -- get hardware inventory. 
  * rvitals -- get hardware vitals. 
  * rcons -- get console 
  * rspconfig -- config node's service processor. 
  * rsetbootseq -- set the boot order. 

etc. 

## Implementation

### Baremetal node discovery

xCAT has several [node discovery](https://sourceforge.net/apps/mediawiki/xcat/index.php?title=XCAT_iDataPlex_Cluster_Quick_Start#Discover_the_Nodes) methods. During the node discovery, the nodes mac address, CPU count, memory size and disk sizes will be discovered and saved in the following xCAT tables: 

  * mac 
  * hwinv (new, see [Hardware_Inventory_for_Node_] for details) 

The node's BMC ip address, user name and password are saved in the **ipmi** table. Once nodes are discovered, the node will be assigned a static ip address on the xCAT management network and the DHCP lease will be setup for the node so that it is ready for deployment. 

### Baremetal node registration

A nova baremetal node registration command takes several node attributes, for example: 
    
     nova baremetal-node-create \
         --pm_address=10.1.0.1 \
         --pm_user=userid \
         --pm_password=password \
         myhost \
         2 4096 286 \
         34:56:78:90:AB:CD
    

It takes the following attributes as the input: 

  * BMC ip addresss, user id and password 
  * Name of nova compute host which will control this baremetal node 
  * Number of CPUs in the node 
  * Memory in the node (MB) 
  * Local hard disk in the node (GB) 
  * MAC address to provision the node 

A new command called **opsaddbmnode** will pull the above baremetal node information from xCAT tables and call "nova baremetal-node-create" to register the baremetal node with the OpenStack cloud. The syntax of the command is: 
    
     opsaddbmnode &lt;noderange&gt; -s &lt;service_host&gt;
       the _service_host_ is the name of the OpenStack compute host that is hosting the baremetal nodes.
           
     opsaddbmnode[-h|--help]
    

Please make sure the following xCAT tables are filled with correct information for the given nodes before calling this command. 

  * ipmi (for BMC ip addresss, user id and password) 
  * mac (for MAC address) 
  * hwinv (for CPU, memory and disk info.) 

An error will be given if the attributes cannot be found in these xCAT tables. 

### Image creation and registration

xCAT supports stateless, stateful and statelite images. The admin can help the user create images using the xCAT. Please see [Deploying Nodes](https://sourceforge.net/apps/mediawiki/xcat/index.php?title=XCAT_iDataPlex_Cluster_Quick_Start#Deploying_Nodes) for details. 

A new command called **opsaddimage** will be used to register the xCAT images into the OpenStack glance tables. The following is the syntax of the command: 
    
      opsaddimage &lt;image1,image2...&gt; [-n &lt;new_name1,new_name2...&gt;] -c &lt;cloud_name&gt;
            The _new_name_ is the OpenStack image name 
            for this image. It defaults to the xCAT image name.
            The _cloud_name_ can be found from the xCAT clouds table.
            
      opsaddimage[-h|--help]
        
    

Under the cover the following glance command will be called: 
    
      cd /tmp
      touch my-image.qcow2
      glance image-create --name &lt;xcat_image_name&gt; --public --disk-format qcow2 \
        --container-format bare \
        --property xcat_image_name=&lt;xcat_image_name&gt; &lt;flavor_id&gt; --image &lt;image_name&gt; mynode
         for FLAT network
      
      nova boot --flavor &lt;flavor_id&gt; --image &lt;image_name&gt; --nic net-id=&lt;net_id&gt;
         for per-tenant based network
    

A nova compute host (node) will be setup. It can be a separate node or on the OpenStack controller node. xCAT will supply a python module that implements the **nova.virt.ComputeDriver** called **xCATBareMetalDriver**. The xCATBareMetalDriver will be installed on the compute node and /etc/nova/nova.conf file will be changed to indicate that this special driver will be used for the compute node. 
    
       compute_driver = xcat.openstack.baremetal.driver.xCATBareMetalDriver
    

xCATBareMetalDriver replaces nova's BaremetalDriver in that it uses xCAT to do the baremetal node deployment. The compute node must have **xCAT-client** installed in order to call xCAT commands from the xCATBareMetalDriver. xCATBareMetalDriver will implement the a few nova.virt.ComputeDriver interfaces: 

  * spawn: 
    * From the input parameters, it derives the xCAT image name, xCAT node name. 
    * It gets the new ip address, subnet and netmask for the node. if the ip given by the OpenStack is in the same subnet as the xCAT management subnet, it will call xCAT command **chdef** and **makehosts** to substitue the xCAT's ip with the OpenStack ip for the node in xCAT DB. The node will be booting up using this new IP. Otherwise, the node will be booting up with xCAT's original static ip and a postbootscript will add the OpenStack ip as an alias after the node is up. 
    * It then calls xCAT command **nodeset noename osimage=&lt;imagename&gt;** to prepare the tftp and dhcp for the node deployment. 
    * It adds a postbootscript called **config_ops_bmnode**. 
    * It calls xCAT command **rpower boot** to start the node deployment. 
    * After the first boot, the **config_ops_bmnode** postbootscript will configure the baremetal node to have the correct ip and host name. 
    * It updates the node status for OpenStack. 
  * reboot: It reboot the baremetal node by calling xCAT command **rpower boot**
  * destroy: 
    * It calls xCAT command **rpower off** to turn the node off. 
    * It it removes the **config_ops_bmnode** from the postbootscripts for the node. 
    * It updates the OpenStack node status. 
  * power_off: It calls xCAT command **rpower off**. 
  * power_on: It calls xCAT command **rpower on**. 
  * get_console_output: (tbd) 
  * get_info: It calls xCAT command "rpower stat". 

**config_ops_bmnode**

This is a new xCAT postscript. It takes a hostname, ip(optional), and netmast(optional) as input. It will change the node hostname. If the ip is present, it will add this ip/netmask to the install nic as an alias. 

**Security**

By default, xCAT creates ssh credentials on the nodes so that the nodes can ssh to each other without password as root. For security reasons, especially for the per-tenant network, we'll disable this feature by the site table attribute: **site.sshbetweennodes=NOGROUPS**. Further study needs to be done to see if remoteshell and other postscripts are needed. 

## Packaging

All code will be put in xCAT-OpenStack package. 