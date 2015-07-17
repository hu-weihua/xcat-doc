The following table is a summary of the Operating System versions and Hardware specifically tested by the xCAT development team and the version of xCAT that they were first made available. Other operating systems and hardware may work with xCAT but have not been explicitly tested by this team. Also, some key new functions are listed for each release. For a complete list of new functions, bug fixes, restrictions, and known problems, refer to the individual release notes. 


**xCAT 2.8 Tested Operating Systems and Hardware Support**


xCAT Version | New OS Support | New Hardware Support | Release Notes | Some key new functions 
------------ | -------------- | -------------------- | ------------- | -----------------------
**xCAT 2.8.4 May 23, 2014** | RHEL 6.5 RHEL 5.10 |  | [XCAT_2.8.4_Release_Notes] | RHEL 7 experimental, support xCAT cluster zones, various command enhancements 
**xCAT 2.8.3 Nov 15, 2013** | AIX 7.3.1.1 AIX 7.3.1.0 AIX 7.1.2 | Xeon Phi (phase2) NeXtScale nx360 M4 | [XCAT_2.8.3_Release_Notes] | xcatd flow control, sysclone x86_64 image provisioning on most OS's, genitird command and nodeset updates,deploy OpenStack on ubuntu,various command enhancements,confignics enhancements,sequential discovery enhancements,kit enhancements 
**xCAT 2.8.2 Jun 26, 2013** | SLES 11 SP3 | Xeon Phi (limited) | [XCAT_2.8.2_Release_Notes] | sysclone x86_64 image provisioning on RHEL and CentOS, precreate postscripts for AIX, kit enhancements, HPC kits for ppc64, ubuntu enhancements, xdsh and updatenode enhancements,sequential discovery enhancements,use of local disk for stateless,Deploy OpenStack cloud 
**xCAT 2.8.1 May 10, 2013** | RHEL 6.4   RHEL 5.9 | | [XCAT_2.8.1_Release_Notes] | **added AIX and RHEL5 for xCAT 2.8**, energy management for Flex,sequential discovery,xCAT Software Kits,Stateful images for MN,osimage enhancements,IPv6 enhancements,ubuntu enhancements,*def enhancements,xdsh and xdcp and updatenode enhancements 
**xCAT 2.8 Feb 28, 2013** | ubuntu 12.04   Windows Server 2012   Windows 8   Hyper-V | | [XCAT_2.8_Release_Notes] | **Based on xCAT 2.7.5**, **Linux RHEL6 and SLES only, no AIX or RHEL5**,Preliminary xCAT Software Kits,multiple hostname domains,x Flex IMM setup,Windows support enhancements,KVM enhancements,z/VM enhancements,virtualization with RHEV,statelite local scratch disk,MN as a managed node,site auditskipcmds,precreate postscripts for Linux,mypostscript templates,pasu command,run postscripts on stateful boot,node update status attrs,updatenode enhancements, deprecate nodeset provmethods and use osimage instead,deprecate bind and use ddns instead


**xCAT 2.7 Tested Operating Systems and Hardware Support**

xCAT Version | New OS Support | New Hardware Support | Release Notes | Some key new functions 
------------ | -------------- | -------------------- | ------------- | ---------------------
**xCAT 2.7.8 Jan 24, 2014** | AIX 7.1.3.1   AIX 7.1.3.0   AIX 6.1.9.1 | | [XCAT_2.7.8_Release_Notes] |
**xCAT 2.7.7 May 17, 2013** | RHEL 6.4 | | [XCAT_2.7.7_Release_Notes] | sinv for devices,Flex energy mgt and rbeacon 
**xCAT 2.7.6 Nov 30, 2012 ** | SLES 10 SP4    AIX 6.1.8    AIX 7.1.2 | | [XCAT_2.7.6_Release_Notes] | HPC Integration updates 
**xCAT 2.7.5 Oct 29, 2012 ** | RHEL 6.3 | | [XCAT_2.7.5_Release_Notes] | virtualization with RHEV,hardware discovery for x Flex,enhanced AIX HASN 
**xCAT 2.7.4 Aug 27, 2012** | SLES 11 SP2 | system x Flex | [XCAT_2.7.4_Release_Notes] | improved IPMI for large systems 
**xCAT 2.7.3 Jun 22, 2012** | SLES 11 SP2 (limited) system x Flex (RHEL 6.2) | | [XCAT_2.7.3_Release_Notes] | HPC Integration updates 
**xCAT 2.7.2 May 25, 2012** | AIX 7.1.1.3 | p775, system p Flex | [XCAT_2.7.2_Release_Notes] | SLES 11 kdump,HPC Integration updates 
**xCAT 2.7.1 Apr 20, 2012** | | | [XCAT_2.7.1_Release_Notes] | bug fixes and minor enhancements 
**xCAT 2.7 Mar 19, 2012** | RHEL 6.2 | 'M4' generation hardware Mellanox IB QDR | [XCAT_2.7_Release_Notes] | xcatd memory usage reduced by two-thirds,xcatdebug command for xcatd and plugins,lstree command,x86_64 genesis boot image,ipmi throttles,rpower suspend select IBM hw,stateful ESXi5,xnba UEFI boot,httpd for postscripts,rolling updates,Nagios monitoring plugin