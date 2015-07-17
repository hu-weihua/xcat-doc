<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [Introduction](#introduction)
  - [Overview](#overview)
    - [P775 Cluster Overview](#p775-cluster-overview)
  - [P775 Fail in Place (FIP)](#p775-fail-in-place-fip)
    - [P775 Octant Failures](#p775-octant-failures)
  - [P775 Recovery Commands and Tasks](#p775-recovery-commands-and-tasks)
  - [P775 FIP xCAT Administrator scenarios](#p775-fip-xcat-administrator-scenarios)
    - [P775 FIP xCAT Administrator Scenario](#p775-fip-xcat-administrator-scenario)
  - [P775 FIP Compute Node Implementation](#p775-fip-compute-node-implementation)
    - [Hot and Warm compute node scenario](#hot-and-warm-compute-node-scenario)
    - [Cold compute node scenario](#cold-compute-node-scenario)
  - [P775 FIP Login Node Implementation](#p775-fip-login-node-implementation)
    - [Login node Replace Ethernet/HFI scenario](#login-node-replace-ethernethfi-scenario)
  - [P775 FIP xCAT Service Node Implementation](#p775-fip-xcat-service-node-implementation)
    - [xCAT SN Ethernet/HFI scenario in same CEC](#xcat-sn-ethernethfi-scenario-in-same-cec)
    - [xCAT SN Disk replacement scenario on a Different CEC](#xcat-sn-disk-replacement-scenario-on-a-different-cec)
  - [FIP GPFS Node Implementation](#fip-gpfs-node-implementation)
    - [GPFS FC/SAS scenario](#gpfs-fcsas-scenario)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Introduction

**Note: This document is no longer updated. Refer to** [Power_775_Cluster_Recovery] for current information. ****

## Overview

This xCAT section provides information about xCAT cluster recovery. It will introduce xCAT commands and procedures that will be used by the xCAT admin to manage the cluster for serviceability. The initial work for this document will focus on our xCAT support for P775 cluster, but we will include recovery scenarios for other xCAT clusters in the future. 

### P775 Cluster Overview

The P775 cluster has a unique cluster service support environment that requires special setup and recovery. We advise that the xCAT administrator to reference "High Performance clustering using the 9125-F2C" Cluster guide to gain more detail about the P775 hardware and service capabilities. This section will introduce xCAT information about P775 Fail In Place (FIP) support. It also provide a sections about different serviceability scenarios that the xCAT administrator may perform when trying to recover from hardware failures. 

There is a separate P775 HW/SW Hardware Service Procedures document which is being used to track various service recovery procedures that will be executed on the P775 cluster. This document will be used by the IBM Service teams to walk through the proper recovery procedures by Hardware teams. Some of these activities will require some execution by the xCAT administrator. We will try and provide step by step procedures for some of the critical xCAT administrator work activities related to hardware recovery. 

## P775 Fail in Place (FIP)

The P775 Fail In Place (FIP) is a policy with P775 hardware that allows to keep selected failed hardware resources in place on the overall P775 cluster for the duration of the maintenance contract in a given customer deployment. In a large cluster, when a hardware failure occurs in the system, a recovery action of some type takes place to allow the system to continue to function without the failing hardware. 

The Toolkit for Event Analysis and Logging (TEAL) product will monitor events and log failures using the Service Focal Point (SFP) through the HMC, and Integrated Switch Network Manager (ISNM) on the xCAT EMS with TEAL listeners. It will log all hardware failures and events in TEAL tables that are located in the xCAT data base on the xCAT EMS. The xCAT administrator can use TEAL commands to reference the different events and alerts found in the P775 cluster. The admin should reference the TEAL and the ISNM documentation that is provided in the "High Performance clustering using the 9125-F2C" Cluster guide for more information. 

The Product Engineer(PE) is the IBM Hardware Service representative for the P775 cluster, and there is the IBM service support team (SWAT) that will help evaluate whether the current FIP information has reached the P775 cluster threshold. They will analyze hardware failures and the current status for the P775 Frame, CECs, and Octants. They will work with the P775 customer,and xCAT administrator to provide the proper service plan to replace or move P775 CEC and Octant FIP resources. 

### P775 Octant Failures

The FIP environment is specified as the FIP Swap policy as hot, warm, and cold. The "hot" swap policy is where they can use the extra FIP nodes as part compute processing, which provides additional processing power for production or non production activities. The "warm" policy is specified where the FIP nodes are powered up and made available to the P775 cluster, but they are not used with any production work load. The "cold" swap policy is where the FIP node resources are physically powered down and are brought online when required. The xCAT administrator with help of IBM service will need to make a decision on how they want to enable the FIP resources, where they may be able to allocate some FIP resources into all three swap policies. The xCAT admin needs to keep track of the FIP resources. It would be get to place these FIP based available octants in a xCAT node groups such as fip_available. then place any P775 failed octants in a FIP defective node group such as fip_defective. 

The administrator needs to understand the type of P775 octant (node), and the type of failure found. There are different recovery procedures identified based on which P775 resource has failed, and the type of P775 octant is affected. The type of P775 octants are known as compute, login, xCAT Server Node (SN), and GPFS I/O Server nodes. 

The P775 cluster is mostly populated with compute octants, where the LPARs are diskless and contain system resources for HFI, memory and CPUs. There are certain rules that xCAT administrator will follow to specify when the compute node octant should be considered as failed. For the compute node octant failures, the LoadLeveler (LoadL) product will check the state of the compute node failures, and will remove the failed nodes from the LoadL resource group. These failed octants will not scheduled any LoadL jobs to failed octants. The xCAT administrator will power off the failed LPAR node, and place the octant in FIP failed "fip_defective" node group. 

The login node allows customers to have access into the P775 cluster. This login node contains HFI, memory, CPUs, but will also contain an ethernet I/O interface for the outside login capability. These login nodes are not listed in the LoadL resource compute group, but do have connectivity to LoadL to schedule and monitor LoadL jobs. 

The xCAT SN nodes are very important in the P775 cluster, since they contain the most I/O resources. They are used to install and manage the diskless compute nodes, and they actively communicate back to the xCAT EMS. The xCAT SN node contains HFI, memory, CPUs, and most of the I/O resources (ethernet, disks) on the P775 CEC. Each xCAT SN should have a backup xCAT SN that can be used if there are software or hardware resource failures that make the xCAT SN unresponsive. The xCAT administrator will follow a xCAT SN failover procedure so the P775 compute nodes can continue to work from the xCAT SN backup node. The xCAT administrator needs to actively debug the xCAT SN failure, and try to get it working as soon as possible. If the xCAT SN has failures that make the octant be listed as FIP failed node, they will need to allocate the required I/O resources to a different FIP octant in the P775 cluster. We will provide selected xCAT SN fail over and FIP based procedures later in this document. 

The GPFS I/O Server node is predominantly used to connect the Disk Extension Controller resources in the P775 cluster to a GPFS cluster. The P775 GPFS I/O Server node contains the HFI, memory, CPU, and selected SAS/FC I/O resources that provide connectivity to the Disk Extension drawers and GPFS storage. There are TEAL GPFS collectors that keep track of the disk I/O events provided by the GPFS subsystems. 

For non compute octant failures with xCAT SN and GPFS I/O server nodes, the administrators may need to move PCI cards to a different P775 octant based on the type of octant failure. If the current P775 CEC does not contain a functional FIP octant in the same P775 CEC, the IBM PE service rep may need to move PCI cards to a FIP node in a different P775 CEC. This will require the xCAT administrator to logically swap the LPAR node of the failed octant to now take over the P775 octant location information of the FIP octant in the xCAT DB. The first attempt should be to swap a bad octant to a FIP octant in the same P775 CEC since the xCAT administrator only needs to power down one P775 CEC for its recovery. If that's not possible, and we need to use a different P775 CEC, the xCAT administrator will need to power down both P775 CECs as part of the recovery. 

There are 2 phases of Fail In Place (FIP) support. The first phase of FIP requires the administrator to do most of the recovery operations using TEAL, LoadL, GPFS and xCAT commands along with step by step procedures. The second phase of P775 FIP support in future P775 releases will provide more automatic recovery activities, based on P775 octant failures. 

## P775 Recovery Commands and Tasks

The xCAT administrator will execute commands for LoadL, Teal, GPFS, and xCAT. It is important the admin reference the man pages for each of the commands being used with FIP tasks. We have provided an overview of the xCAT commands that will be used with FIP environment here. 
    
    swapnodes  - swap the I/O and location information in the xCAT db between two nodes. This command will assign the
    IO adapters from the bad octant over to the specified good FIP octant, where the current LPAR node name is not changed. 
    This would mean that any pertinent I/O assignments is still contained in the current node object, but the octant LPAR id,
    and possibly MTMS information is swapped.  The failed octant is then listed as a bad FIP node and should not be referenced 
    until it is repaired. The swapnodes command will make updates for the ppc and nodepos table attributes. It is very important
    that the PE rep communicates what the real physical octant location when I/O resources are moved to the new FIP location. 
    
    
    chvm/lsvm  - For Power 775, chvm is used to change the octant configuration values for the I/O slots assignment to LPARs  
    within the same CEC. The administrator should use lsvm to get the profile content as a file, edit the content file, and 
    node name with ":" manually before the I/O information which will be assigned to each node. The profile content can be piped 
    into the chvm command, or modified with the chvm -p flag.
    
    
    rinv    - retrieves hardware configuration information for firmware and deconfigured resources from the Service Processor 
    for P775 CECs. The administrator may want to check and see if there are any deconfigured resources for each P7 CEC node objects 
    
    
    gatherfip - script used by xCAT admin gather FIP and service events from TEAL. This script places current service events for 
    ISNM, SFP, and deconfigured resources into files and a tar package in /var/log directory which can sent to IBM service team.  
     Writing CNM Alerts in TEAL to  file /var/log/gatherfip.CNM.Events
     Writing CNM Link Down information to /var/log/gatherfip.CNM.links.down
     Writing deconfigred resource information to /var/log/gatherfip.guarded.resources
     Writing Hardware service events information to /var/log/gatherfip.hardware.service.events
     Created compressed tar FIP based file /var/log/&lt;xCAT EMS&gt;.gatherfip.&lt;date_time&gt;.tar.gz  
    
    
     lsdef   -  the lsdef command is used the list attributes for xCAT table definitions. For FIP and service recovery, it is  
     important to know the VPD information that is listed for the Frame and CECs in the P775 cluster. The important attributes 
     for vpd are  hcp (hardware control point), id (frame id # or cage id #), mtm (model type, machine) serial (serial #). 
     The lsdef command is also used to validate the P775 LPAR or octant information. The important attributes listed with 
     LPARs are cons (console), hcp (hardware control point), id (LPAR/octant id #), mac (Ethernet or HFI MAC address),
     parent (CEC object), os (operating system), xcatmaster (installation server xCAT SN or xCAT EMS).          
     
     To gather information about Frame objects 
       lsdef frame -i hcp,id,mtm,serial
      Object name: frame17
       hcp=frame17
       id=17
       mtm=9458-100
       serial=BPCF017 
    &gt;
     To gather information about CEC objects
       lsdef f17c04  -i hcp,id,mtm,serial
      Object name: f17c04
       hcp=f17c04
       id=6
       mtm=9125-F2C
       serial=02D8B55
    &gt;  
     To gather information about LPARs (octants), you need to reference different attributes.
       lsdef c250f03c01ap01-hf0 -i cons,hcp,id,mac,os,parent,xcatmaster
      Object name: c250f03c01ap01-hf0
       cons=fsp
       hcp=cec01
       id=1
       mac=020000000004
       os=AIX
       parent=cec01
       xcatmaster=c250f03c10ap01-hf0
    

  

    
     rpower   -  this command is used at times to logically power off/on LPARs (octants), CEC (fsp), and frame (bpa).
     It has different attributes based on the hardware type of the node object being targeted.   
      For Frame objects: rpower noderange [rackstandby|exit_rackstandby|stat|state]
      For CEC objects:  rpower noderange [on|off|stat|state|lowpower]
      For LPAR objects: rpower noderange [on|off|stat|state|reset|boot|of|sms] 
    

  

    
     mkhwconn/rmhwconn -   these commands create and removes hardware connections for FSP and BPA nodes to HMC nodes or
     hardware server from the xCAT EMS. As part of service recovery, you may need to remove and recreate HW connections to 
     if there were changes being made to a BPA or FSP.
    

## P775 FIP xCAT Administrator scenarios

This section provides the expected xCAT administrator tasks that is required for FIP activities. The expectation is that this section will continue to be updated with additional scenarios when more service and recovery information is known and FIP and service activities. 

  


### P775 FIP xCAT Administrator Scenario

The xCAT administrator should keep a close watch in regard to P775 cluster activities. These include keeping track of any P775 cluster resources and understanding if there are any hardware, network, and software issues found. It is important that the P775 admin read through the "High Performance Clustering using the 9125-F2C" (P775 Cluster) guide , and gain a good understanding the TEAL and ISNM infrastructure and commands used to track the P775 hardware and HPC software events. These include P775 hardware events from the Service Focal Point (SFP), that are created on the Hardware Management Console (HMC), and then placed in the Teal tables in the xCAT DB on the xCAT EMS. The P775 admin should also understand the ISNM commands used to manage the HFI switch environment, where there are ISNM tables placed in the xCAT DB and configuration files placed on the xCAT EMS that will track the P775 HFI configuration and link errors. 

The administrator should setup xCAT FIP node groups that should be used working with FIP environment. There should be one xCAT node group called "fip_defective" for any found FIP defective nodes or octants. There should be a second xCAT node group "fip_available" that should list the FIP available nodes or octants. You can use the xCAT mkdef command to create the node groups, and then use the chdef command to associate any node (octants) to the proper node group. 
    
    mkdef -t group -o fip_defective  (create a fip_defective group that should be empty)
    mkdef -t group -o fip_available members="node1,node2,node3" (create a fip_available group with nodes node1,node2node3)
      
    

There is an xCAT script called "gatherfip" that is installed on the xCAT EMS that will track many of the P775 hardware and ISNM events. This script is available to the xCAT administrator to execute on the xCAT EMS where it will create files for ISNM, SFP, and deconfigured resources into files and a tar package in /var/log directory which can sent to IBM service team. 
    
     # gatherfip  
     Writing CNM Alerts in TEAL to  file /var/log/gatherfip.CNM.Events
     Writing CNM Link Down information to /var/log/gatherfip.CNM.links.down
     Writing deconfigred resource information to /var/log/gatherfip.guarded.resources
     Writing Hardware service events information to /var/log/gatherfip.hardware.service.events
     Created compressed tar FIP based file /var/log/&lt;xCAT EMS&gt;.gatherfip.&lt;date_time&gt;.tar.gz  
    

The xCAT administrator may want check the current state of P775 frame, cecs, and lpars (octants). There are xCAT commands that can be used to track hardware configuration information. Here is some information for important xCAT commands rinv, rpower, lsdef, and lsvm that will provide current status of P775 cluster. 

The rinv command retrieves hardware configuration information for powercode (frame) and firmware (cec) levels using "firm" attribute. It also specifies any deconfigured resources with "deconfig" attributes from the FSP for the P775 CECs. 
    
     #rinv frame03 firm        (P775  frame)
      frame03: Release Level  : 02AP730
      frame03: Active Level   : 033
      frame03: Installed Level: 033
      frame03: Accepted Level : 032
      frame03: Release Level A: 02AP730
      frame03: Level A        : 033
      frame03: Current Power on side A: temp
      frame03: Release Level B: 02AP730
      frame03: Level B        : 033
      frame03: Current Power on side B: temp
     # rinv cec01 firm        (P775 CEC)
     cec01: Release Level  : 01AS730
     cec01: Active Level   : 035
     cec01: Installed Level: 035
     cec01: Accepted Level : 034
     cec01: Release Level Primary: 01AS730
     cec01: Level Primary  : 035
     cec01: Current Power on side Primary: temp
     cec01: Release Level Secondary: 01AS730
     cec01: Level Secondary: 035
     cec01: Current Power on side Secondary: temp 
     # rinv cec01 deconfig   (P775 CEC)
     cec01: Deconfigured resources
     cec01: Location_code                RID   Call_Out_Method    Call_Out_Hardware_State    TYPE
     cec01: U78A9.001.1147006-P1         800
    

The rpower command is used to logically power off/on and track the state of P775 LPARs (octants), CEC (fsp), and frame (bpa). It has different attributes based on the hardware type of the node object(s) being targeted. 
    
     For Frame objects: rpower noderange [rackstandby|exit_rackstandby|stat|state]
     For CEC objects:  rpower noderange [on|off|stat|state|lowpower]
     For LPAR objects: rpower noderange [on|off|stat|state|reset|boot|of|sms] 
     
     # rpower frame03 state    (P775 Frame)
     frame03: BPA state - Both BPAs at standby 
     # rpower cec01  state     (P775 CEC)
     cec01: operating
     # rpower c250f03c01ap01-hf0 state   (P775 LPAR)
     c250f03c01ap01-hf0: Running
    

The lsdef command is used the list attributes for xCAT table definitions. For FIP and service recovery, it is important to know the VPD information that is listed for the Frame and CECs in the P775 cluster. The important attributes for vpd are hcp (hardware control point), parent, id (frame id/cage id), mtm (model type machine) and serial number. 
    
     To gather hcp and vpd information about Frame objects 
     # lsdef frame03 -i hcp,id,mtm,serial,parent   (P775 Frame)
     Object name: frame03
       hcp=frame03
       id=3
       mtm=78AC-100
       parent=
       serial=BD50095
    &gt; 
     To gather hcp and vpd  information about CEC objects
     # lsdef cec01 -i hcp,id,mtm,serial,parent  (P775 CEC)
     Object name: cec01
       hcp=cec01
       id=3
       mtm=9125-F2C
       parent=frame03
       serial=02A5CE6   
    

For xCAT LPARs(octants), it is important to reference a different set of attributes. The important xCAT attributes are hcp, cons, lpar/octant id, MAC address, OS, parent, and xcatmaster (install server). To gather vpd information about LPARs (octants) you need to reference the CEC node object listed as the parent. 
    
     # lsdef c250f03c01ap01-hf0 -i cons,hcp,id,mac,os,parent,xcatmaster  (P775 diskless LPAR)
     Object name: c250f03c01ap01-hf0
       cons=fsp
       hcp=cec01
       id=1
       mac=020000000004
       os=AIX
       parent=cec01
       xcatmaster=c250f03c10ap01-hf0
    

  
The lsvm command is used to list the octant configuration and I/O slots assignment for P775 CECs and LPARs. The administrator should use lsvm to get the profile content information to help track any I/O assignmemnt changes being made to LPAR or CEC. This is very important to track the octant information for the xCAT SN when there are changes required with FIP nodes. 
    
     # lsvm cec10  (P775 CEC with I/O resources)
       1: 569/U78A9.001.114M005-P1-C1/0x21010239/2/1
       1: 568/U78A9.001.114M005-P1-C2/0x21010238/2/1
       1: 561/U78A9.001.114M005-P1-C3/0x21010231/2/1
       1: 560/U78A9.001.114M005-P1-C4/0x21010230/2/1
       1: 553/U78A9.001.114M005-P1-C5/0x21010229/2/1
       1: 552/U78A9.001.114M005-P1-C6/0x21010228/2/1
       1: 545/U78A9.001.114M005-P1-C7/0x21010221/2/1
       1: 544/U78A9.001.114M005-P1-C8/0x21010220/2/1
       1: 537/U78A9.001.114M005-P1-C9/0x21010219/2/1
       1: 536/U78A9.001.114M005-P1-C10/0x21010218/2/1
       1: 529/U78A9.001.114M005-P1-C11/0x21010211/2/1
       1: 528/U78A9.001.114M005-P1-C12/0x21010210/2/1
       1: 521/U78A9.001.114M005-P1-C13/0x21010209/2/1
       1: 520/U78A9.001.114M005-P1-C14/0x21010208/2/1
       1: 514/U78A9.001.114M005-P1-C17/0x21010202/2/1
       1: 513/U78A9.001.114M005-P1-C15/0x21010201/2/1
       1: 512/U78A9.001.114M005-P1-C16/0x21010200/2/1
       cec10: PendingPumpMode=1,CurrentPumpMode=1,OctantCount=8:
       OctantID=0,PendingOctCfg=2,CurrentOctCfg=2,PendingMemoryInterleaveMode=2,CurrentMemoryInterleaveMode=2;
       OctantID=1,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
       OctantID=2,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
       OctantID=3,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
       OctantID=4,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
       OctantID=5,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
       OctantID=6,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
       OctantID=7,PendingOctCfg=1,CurrentOctCfg=1,PendingMemoryInterleaveMode=0,CurrentMemoryInterleaveMode=0;
     # lsvm c250f03c10ap01  (P775 xCAT SN LPAR)
       1: 569/U78A9.001.114M005-P1-C1/0x21010239/2/1
       1: 568/U78A9.001.114M005-P1-C2/0x21010238/2/1
       1: 561/U78A9.001.114M005-P1-C3/0x21010231/2/1
       1: 560/U78A9.001.114M005-P1-C4/0x21010230/2/1
       1: 553/U78A9.001.114M005-P1-C5/0x21010229/2/1
       1: 552/U78A9.001.114M005-P1-C6/0x21010228/2/1
       1: 545/U78A9.001.114M005-P1-C7/0x21010221/2/1
       1: 544/U78A9.001.114M005-P1-C8/0x21010220/2/1
       1: 537/U78A9.001.114M005-P1-C9/0x21010219/2/1
       1: 536/U78A9.001.114M005-P1-C10/0x21010218/2/1
       1: 529/U78A9.001.114M005-P1-C11/0x21010211/2/1
       1: 528/U78A9.001.114M005-P1-C12/0x21010210/2/1
       1: 521/U78A9.001.114M005-P1-C13/0x21010209/2/1
       1: 520/U78A9.001.114M005-P1-C14/0x21010208/2/1
       1: 514/U78A9.001.114M005-P1-C17/0x21010202/2/1
       1: 513/U78A9.001.114M005-P1-C15/0x21010201/2/1
       1: 512/U78A9.001.114M005-P1-C16/0x21010200/2/1
    

## P775 FIP Compute Node Implementation

The FIP activity with compute nodes/octants is to keep track of the different hardware issues found with compute nodes. IBM will provide the customer more P775 compute nodes than they paid for as part of FIP, and has choices in regard to how they use the FIP nodes. The FIP environment is specified as the FIP Swap policy as hot, warm, and cold. The "hot" swap policy is where they can use the extra FIP nodes as part compute processing, which provides additional processing power for production or non production activities. The "warm" policy is specified where the FIP nodes are powered up and made available to the P775 cluster, but they are not used with any production work load. The "cold" swap policy is where the FIP node resources are physically powered down and are brought online when required. The xCAT administrator with help of IBM service will need to make a decision on how they want to enable the FIP resources, where they may be able to allocate some FIP resources into all three swap policies. They will need to manually keep track on how the FIP resources are being allocated for their application workloads. 

When the compute node has lost CPU/memory resources or HFI link connections where the node is no longer appropriate to be used as a compute node for LoadLeveler application activity. The xCAT administrator will need to remove the bad octant (node) from the LoadL resource pool so the node is no longer scheduled for any applications. Depending on the swap policy, the admin will decide which action is required. 
    
     - hot swap policy   -  the administrator just removes the bad node from the LoadL resource pool and places the bad 
       octant/node in the FIP failed node group fip_defective. 
     - warm swap polify  -  the administrator removes the bad node from the LoadL resource pool, places the bad octant/node  in the
       FIP failed node group fip_defective. They may place setup FIP available compute nodes to be included in LoadL resource pool.
     - cold swap policy  - the administrator removes the bad node from the LoadL resource pool, places the bad octant/node  in the
       FIP failed node group fip_defective. They need to power up the CEC/octant, setup and install FIP available nodes, then place
       the compute nodes in LoadL resource pool    
    

### Hot and Warm compute node scenario
    
    TBD  Need  LoadL  and xCAT  commands   
    

  


### Cold compute node scenario
    
    TBD  Need  LoadL  and xCAT  commands (include power up, and compute node installation)
    

## P775 FIP Login Node Implementation

The FIP activity with P775 Login nodes/octants is to make sure that we have proper FIP nodes available in the P775 CEC that contains an available ethernet adapter, and is available to the same xCAT SN. As with compute nodes, the login nodes are diskless, so they contain octant resoureces of HFI, CPU and memmory. But they do need to have and ethernet I/O resource be included in the octant configuration. There are multiple P775 login nodes in the cluster, so the plan is that the P775 administrator will instruct the users to use one the other login nodes, while they rebuild a new login node from an available FIP resource. Based on the login node failure, the admin will need to locate an FIP octant, and then make sure the I/O ethernet adapter resource using xCAT "chvm" to a new designated FIP node octant. They will execute xCAT swapnodes command where you place the bad octant into fip_defective node group, and aloocate new octant as the new login node definition. The admin will need to see if the new login node requires a new ethernet "MAC" address to be used, and they will install the new P775 login node from the appropriate xCAT SN with same OS diskless image that was used with the previous login node used. 

### Login node Replace Ethernet/HFI scenario
    
    TBD  Need  LoadL  and xCAT  commands
    

## P775 FIP xCAT Service Node Implementation

The FIP activity with P775 xCAT SN nodes/octants is to make sure that we have proper FIP nodes available in the P775 CEC that contains an available I/O resources including an ethernet adapter, and the SAS disk adapters. Since the xCAT SN is very prominent in the P775 cluster, the administrator needs to actively recover the xCAT SN very quickly. It would be advantageous to have the available FIP resources made available in the same P775 CEC as the current xCAT SN. This will help the recovery where only one P775 cec will need to be brought down when looking to reorganize the I/O resources to a new FIP available resource. Since the xCAT SN CEC has most of the I/O resources, the complexity of the xCAT SN failure will require additional debug activity, and many additional xCAT administrator tasks to recover the xCAT SN. 

The first task is to make sure that the current backup xCAT SN is working in the P775 cluster, and admin will execute the xCAT SN failover tasks, where the xCAT compute nodes are allocated over to the xCAT SN backup node. After you accomplish the recovery of the failed xCAT SN, you will then want to reallocate the xCAT compute nodes back to their primary xCAT SN. This xCAT SN failover scenario is documented in the appropriate xCAT AIX/Linux Hierarchical Cluster documents. 
    
[Setting_Up_a_Linux_Hierarchical_Cluster]
[Setting_Up_an_AIX_Hierarchical_Cluster] 
    

The recovery flow of the xCAT SN, is to first debug what the issue is of the xCAT SN, where it is best to recover the xCAT SN without walking through the FIP process. But if there is an issue with the P775 xCAT SN octant, the admin will need to work closely with the IBM PE service representative to understand the xCAT SN hardware failure. The FIP recovery is to power down the P775 CECs for both the failed xCAT SN and for the P775 CEC where the FIP available octant is located. The IBM PE will then makes sure the appropriate physical I/O ethernet and disk resources are moved to the new FIP avaialbe octant. The xCAT admin will make sure the I/O ethernet adapter and the SAS disk I/O resources using xCAT "chvm" are allocated to a new designated FIP node octant. They will execute xCAT swapnodes command where you place the bad octant into fip_defective node group, and aloocate new octant as the new xCAT SN node definition. Based on the xCAT SN hardware failure, the admin may need to do multiple tasks. For a new ethernet adapters, they will need to retrieve new ethernet MAC address for the xCAT SN. For a new replaced SAS disk, the xCAT admin will need to reinstall the xCAT SN using the same xCAT OS disk full image used by the previous xCAT SN octant. If the xCAT SN needed to be moved to a different P775 cec, than the admin needs to make sure the VPD and the remote power attributes are reflected in the xCAT SN object. One the new xCAT SN is properly up and running to the satisfaction of the xCAT administrator they can plan to execute the xCAT SN fail over tasks to move the xCAT compute nodes back to the rebuilt primary xCAT SN. 

  


### xCAT SN Ethernet/HFI scenario in same CEC

This FIP scenario specifies the xCAT admin activity required to move or replace a bad octant being supported as an xCAT SN working in the same P775 CEC. The expectation is that the xCAT admin has noted that there is a failure with both an ethernet adapter and that the octant 0 (LPAR id 1) with xCAT SN "xcatsn1" is not working. The P775 admin has identified an available FIP octant 1 (LPAR id 5) xCAT node "cec1n5" in the same CEC "cec1" can be used to setup 
    
    *  Admin has noted a failure with cec1 octant0 where communication is lost to HFI and ethernet on xCAT SN "xcatsn1".
        It was found that xcatsn1 is seeing decongifured resources in octant 0 for cec1.   
        rinv cec1 deconfig . 
    (1)Admin has executed manual xCAT SN fail over using multiple xCAT commands including "snmove" to have compute nodes use the
       backup xCAT SN. These tasks are defined in the Hierarchical Cluster documentation in section "Using a backup service node". 
[Setting_Up_an_AIX_Hierarchical_Cluster]
[Setting_Up_a_Linux_Hierarchical_Cluster] 
    (2) Admin contacts IBM Service indicating that there is a bad xCAT SN octant where a PE person will be available to
        physically move the I/O resources from octant 0 (LPARid 1) to octant 1 (LPARid 5) in cec1.
    (3)Drain any compute notes found on cec1 and remove the LL resource group. Admin will then power off cec1. 
        ll commands
        rpower cec1  off
    (4) PE Rep will do physical update to cec1 where the ethernet adapter is replaced and, I/O resources allocated to octant 1
       (LPARid 5). The IBM PE service team needs to note which I/O slots resources have been moved.
    (5) Admin executes the xCAT commands swapnode command to have xCAT SN xcatsn1 to now use FIP node fipcec1n5
        octant 1 (lpar id 5) resources in xCAT DB. The FIP node fipcec1n5 will now take ownership of the bad octant 0.
        It is also a good time to make updates to the FIP node groups fip_defective and fip_available for node "fipcec1n5"
          swapnodes -c xcatsn1 -f fipcec1n5
          chdef  -t group  -o fip_defective  members=fipcec1n5
          chdef  -t group  -o fip_available -d members=fipcec1n5
    (6) Admin powers up cec1 to standby so resources can be seen. He executes lsvm cec1 to note octant resources. If changes 
        are needed, admin executes lsvm on xCAT SN to produce output file, then updates file to represent proper I/O setting.
        Admin then executed inputs the file working with chvm command. 
         rpower  cec1 onstandby 
         lsvm    cec1 
         lsvm   xcatsn1 &gt;/tmp/xcatsn1.info
         edit /tmp/xcatsn1.info .. Make updates for octant information, and save file
         cat  /tmp/xcatsn1.info | chvm  xcatsn
         lsvm  xcatsn1
    (7) Admin executes "getmacs"  command to retrieve the new MAC address of the new ethernet adapter. Make sure this MAC address
        is placed in the xcatsn1 node object. The admin will want to recreate xcatsn1 nim object to reflect new MAC interface if
        working with AIX cluster.
          getmacs  xcatsn1 -D
          lsdef xcatsn1 -i mac
          xcat2nim xcatsn1 -f  (AIX only)
    (8) Since the disk subsystem was not affected, there is a good chance that you should beable to power up the xCAT SN and other  
        compute  node octants located on the cec1. The admin should do a thorough checkout making sure all xCAT xCAT SN 
        environments (ssh, DB2, and installation) are working properly.  It is a good test to execute xCAT updatenode command
        against the xCAT SN.  If the xCAT SN is not working properly, the admin may want to do a reinstall on the xCAT SN.
           rpower xcatsn1  on 
           ssh root@xcatsn1   (try to login and validate OS and xCAT commands) 
           updatenode  xcatsn1  
    (9) Once the admin has validated that the xCAT SN xcatsn1 is running properly, they can schedule the appropriate time to execute
        manual xCAT SN fail over task to have the selected compute nodes move from the backup xCAT SN.  This is the same activity
        that is described in step 1  of this scenario .
    

### xCAT SN Disk replacement scenario on a Different CEC

This FIP scenario specifies the xCAT admin activity required to move or replace a bad octant being supported as an xCAT SN working in a different P775 CEC. The expectation is that the xCAT admin has noted that there is a failure with both a disk,and that the octant 0 (LPAR id 1) with xCAT SN "xcatsn1" is not working in "cec1". The P775 admin identified an available FIP octant 0 (LPAR id 1) with xCAT node "fipcec2n1" which is available is a different CEC "cec2" that can be used to setup the xCAT SN. 
    
    *  Admin has noted a failure with cec1 octant0 where communication is lost to HFI and the disk is bad on xCAT SN "xcatsn1".
        It was found that xCAT SN xcatsn1 is seeing decongifured resources in octant 0 for cec1.   
        rinv cec1 deconfig   
    (1)Admin has executed manual xCAT SN fail over using multiple xCAT commands including "snmove" to have compute nodes use the
       backup xCAT SN. These tasks are defined in the Hierarchical Cluster documentation in section "Using a backup service node". 
[Setting_Up_an_AIX_Hierarchical_Cluster]
[Setting_Up_a_Linux_Hierarchical_Cluster] 
    (2) Admin contacts IBM Service indicating that there is a bad xCAT SN octant where a PE person will be available to
        physically move the I/O resources from octant 0 (LPARid 1) to octant 1 (LPARid 5) in cec1.
    (3) Since the admin needs to use the FIP available node on a different CEC,we need to drain any compute nodes found on cec1 and 
        and cec2, and remove them from the LL resource group. Admin will then need to power off cec1 and cec2. 
        ll commands
        rpower cec1,cec2  off
    (4) PE Rep will do physical update to cec1 where the disk is replaced, and xCAT SN required I/O resources ethernet and disk 
        adapters are physically installed and  allocated to octant 0 in cec2. We will use the HFI interfaces in octant 0 in cec2.
        The IBM PE service team needs to note which I/O slots resources have been moved.
    (5) Admin executes the xCAT commands swapnode command to have xCAT SN xcatsn1 to now use FIP node fipcec2n1 settings with cec2
        octant 0 (lpar id 1) resources in xCAT DB. This will indicate that xCAT SN xcatsn1 will have a new VPD and MTMS attributes.
        The FIP node fipcec2n1 will now take ownership of the bad octant 0 in cec1.
        It is also a good time to make updates to the FIP node groups fip_defective and fip_available for node "fipcec2n1"
          swapnodes -c xcatsn1 -f fipcec1n5
          chdef  -t group  -o fip_defective  members=fipcec2n1
          chdef  -t group  -o fip_available -d members=fipcec2n1
    (6) Admin powers up cec1 and cec2 so resources can be seen. He executes lsvm cec2 to note octant resources. If changes 
        are needed, admin executes lsvm on xCAT SN to produce output file, then updates file to represent proper I/O setting.
        Admin then executed inputs the file working with chvm command. 
         rpower  cec1,cec2 on 
         lsvm    cec2 
         lsvm   xcatsn1 &gt;/tmp/xcatsn1.info
         edit /tmp/xcatsn1.info .. Make updates for octant information, and save file
         cat  /tmp/xcatsn1.info | chvm  xcatsn
         lsvm  xcatsn1
    (7) Admin executes "getmacs"  command to validate that proper MAC address of the ethernet adapteris found. Make sure this
        MAC address is placed in the xcatsn1 node object. The admin will want to recreate xcatsn1 nim object to reflect new MAC
        interface if working with AIX cluster.
          getmacs  xcatsn1 -D
          lsdef xcatsn1 -i mac
          xcat2nim xcatsn1 -f  (AIX only)
    (8) Since the disk subsystem was affected, we will need to reinstall the xCAT SN xcatsn1 on the new disk. The admin will
        need to validate all of the service node and installation attributes are properly defined. They will need to executee
        a diskful installation on the xCAT SN. Please reference the proper xCAT SN Hierarchical Cluster documentation.  
        The admin should do a thorough checkout making sure all xCAT xCAT SN environments (ssh, DB2, and installation) are working
        properly after the xCAT SN installation. 
           lsdef xcatsn1       (check all install and SN  attributes)
           rnetboot xcatsn1    (execute network boot to reinstall xcatsn1 on cec2)
           ssh root@xcatsn1   (try to login and validate OS and xCAT commands)         
    (9) Once the admin has validated that the xCAT SN xcatsn1 is running properly, they can schedule the appropriate time to execute
        manual xCAT SN fail over task to have the selected compute nodes move from the backup xCAT SN.  This is the same activity
        that is described in step 1  of this scenario . The admin should plan to reinstall the diskless compute nodes working 
        with the rebuilt xcatsn1. They can also reinstate the good compute nodes in cec1 and cec2 into the LL resources except for
        bad octant 0 in cec1 (fipcec2n1).
    

## FIP GPFS Node Implementation

### GPFS FC/SAS scenario
    
    TBD  Need GPFS and xCAT  commands
    