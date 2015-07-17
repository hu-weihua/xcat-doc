<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
Table of Contents

- [This page shows how to add an updated Broadcom driver module to pxeboot initrd](#this-page-shows-how-to-add-an-updated-broadcom-driver-module-to-pxeboot-initrd)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## This page shows how to add an updated Broadcom driver module to pxeboot initrd
    
    
    
    1: locate kernel and initrd  
    
    [[root@localhost](mailto:root@localhost) centos-4.3-x86_64]# ls -l images/pxeboot/* -h  
    
    -r--r--r-- 1 root root 3.9M Mar 15 21:20 images/pxeboot/initrd.img  
    
    -r--r--r-- 1 root root 274 Mar 15 21:20 images/pxeboot/README  
    
    -r--r--r-- 1 root root 1.7M Mar 15 21:20 images/pxeboot/vmlinuz  
    
    [[root@localhost](mailto:root@localhost) centos-4.3-x86_64]#
    
    Initrd is compressed ext2 filesystem (kernel 2.4) , or compressed cpio archive (kernel 2.6)  
    
    **_For Kernel 2.4:_**  
    
    [[root@localhost](mailto:root@localhost) pxeboot]# file initrd.img  
    
    initrd.img: gzip compressed data, from Unix, max compression
    
    rename it to end in .gz and uncompress  
    
    [[root@localhost](mailto:root@localhost) ~]# gunzip initrd.img.gz  
    
    -r--r--r-- 1 root root 8388608 Mar 27 12:40 initrd.img
    
    [[root@localhost](mailto:root@localhost) ~]# file initrd.img  
    
    initrd.
    
    Let's mount the filesystem:  
    
    [[root@localhost](mailto:root@localhost) ~]# mkdir a  
    
    [[root@localhost](mailto:root@localhost) ~]# mount -o loop initrd.img a  
    
    [[root@localhost](mailto:root@localhost) ~]# cd a  
    
    [[root@localhost](mailto:root@localhost) a]# ls -l  
    
    total 21  
    
    lrwxrwxrwx 1 root root 4 Mar 15 21:20 bin -&gt; sbin  
    
    drwxr-xr-x 4 root root 1024 Mar 15 21:20 dev  
    
    drwxr-xr-x 4 root root 1024 Mar 15 21:20 etc  
    
    lrwxrwxrwx 1 root root 10 Mar 15 21:20 linuxrc -&gt; /sbin/init  
    
    drwx  
    
    
    
    * * *
    
      
    
    2 root root 12288 Mar 15 21:19 lost found  
    
    drwxr-xr-x 2 root root 1024 Mar 15 21:20 modules  
    
    drwxr-xr-x 2 root root 1024 Mar 15 21:20 proc  
    
    drwxr-xr-x 2 root root 1024 Mar 15 21:20 sbin  
    
    drwxr-xr-x 2 root root 1024 Mar 15 21:20 selinux  
    
    drwxr-xr-x 2 root root 1024 Mar 15 21:20 sys  
    
    drwxr-xr-x 2 root root 1024 Mar 15 21:20 tmp  
    
    drwxr-xr-x 5 root root 1024 Mar 15 21:20 var
    
    **_For Kernel 2.6:_**  
    
    [[root@localhost](mailto:root@localhost) pxeboot]# file initrd.img  
    
    initrd.img: gzip compressed data, from Unix, max compression
    
    rename it to end in .gz and uncompress  
    
    [[root@localhost](mailto:root@localhost) ~]# gunzip initrd.img.gz  
    
    -rw-r--r-- 1 root root 7676416 Apr 22 16:03 initrd.img
    
    [[root@xcatsrv1](mailto:root@xcatsrv1) a]# file initrd.img  
    
    initrd.img: ASCII cpio archive (SVR4 with no CRC)
    
    Let's open the compress cpio file:  
    
    [[root@localhost](mailto:root@localhost) ~]# mkdir a  
    
    [[root@localhost](mailto:root@localhost) ~]# cd a  
    
    [[root@localhost](mailto:root@localhost) a]# cpio -dumi   
    
    [[root@localhost](mailto:root@localhost) a]# ls -l
    
    drwxr-xr-x 2 root 252 4096 May 1 2008 tmp  
    
    drwxr-xr-x 2 root 252 4096 May 1 2008 sys  
    
    drwxr-xr-x 2 root 252 4096 May 1 2008 selinux  
    
    drwxr-xr-x 2 root 252 4096 May 1 2008 proc  
    
    drwxr-xr-x 2 root 252 4096 May 1 2008 dev  
    
    drwxr-xr-x 6 root 252 4096 Apr 22 16:05 var  
    
    drwxr-xr-x 2 root 252 4096 Apr 22 16:05 sbin  
    
    drwxr-xr-x 2 root 252 4096 Apr 22 16:05 modules  
    
    lrwxrwxrwx 1 root 252 10 Apr 22 16:05 init -&gt; /sbin/init  
    
    drwxr-xr-x 3 root 252 4096 Apr 22 16:05 etc  
    
    lrwxrwxrwx 1 root 252 4 Apr 22 16:05 bin -&gt; sbin
    
    **_Where are the modules?_**  
    
    [root@localhost](mailto:root@localhost) a]# cd modules/  
    
    module-info modules.cgz modules.dep modules.pcimap  
    
    modules.usbmap pci.ids pcitable
    
    [[root@localhost](mailto:root@localhost) modules]# ls -lh  
    
    total 3.1M  
    
    -rw-r--r-- 1 root root 4.7K Mar 15 21:20 module-info  
    
    -rw-r--r-- 1 root root 2.9M Mar 15 21:20 modules.cgz  
    
    -rw-r--r-- 1 root root 9.7K Mar 15 21:20 modules.dep  
    
    -rw-r--r-- 1 root root 38K Mar 15 21:20 modules.pcimap  
    
    -rw-r--r-- 1 root root 47K Mar 15 21:20 modules.usbmap  
    
    -rw-r--r-- 1 root root 28K Mar 15 21:20 pci.ids  
    
    -rw-r--r-- 1 root root 14K Mar 15 21:20 pcitable
    
    modules.cgz is compressed cpio archive:  
    
    [[root@localhost](mailto:root@localhost) modules]# file modules.cgz  
    
    modules.cgz: gzip compressed data, from Unix, max compression
    
    [[root@localhost](mailto:root@localhost) modules]# cat modules.dep  
    
    orinoco_cs: ds pcmcia_core orinoco hermes  
    
    smc91c92_cs: ds mii pcmcia_core  
    
    vfat: fat  
    
    msdos: fat  
    
    usb-storage: scsi_mod  
    
    8139too: mii  
    
    fmvj18x_cs: ds pcmcia_core  
    
    etc...
    
    Let us uncompress see what is inside:  
    
    [[root@localhost](mailto:root@localhost) modules]# mv /root/modules.cgz /root/modules.cgz.gz  
    
    [[root@localhost](mailto:root@localhost) modules]# gunzip /root/modules.cgz.gz  
    
    [[root@localhost](mailto:root@localhost) modules]# ls -lh /root/modules.cgz  
    
    -rw-r--r-- 1 root root 8.9M Mar 27 12:43 /root/modules.cgz
    
    Extract whatever is inside modules.cgz:  
    
    [[root@localhost](mailto:root@localhost) ~]# cpio -i --verbose --make-directories   
    
    2.6.9-34.EL/x86_64/sky2.ko  
    
    2.6.9-34.EL/x86_64/orinoco_cs.ko  
    
    2.6.9-34.EL/x86_64/smc91c92_cs.ko  
    
    2.6.9-34.EL/x86_64/fat.ko  
    
    2.6.9-34.EL/x86_64/usb-storage.ko  
    
    2.6.9-34.EL/x86_64/8139too.ko  
    
    2.6.9-34.EL/x86_64/fmvj18x_cs.ko  
    
    2.6.9-34.EL/x86_64/sata_sil.ko  
    
    2.6.9-34.EL/x86_64/ips.ko  
    
    etc....
    
    Compile your BCM5721 module:
    
    Note that the installer runs as a UP system, so you need to switch to  
    
    UP kernel when compiling the following:  
    
    [[root@localhost](mailto:root@localhost) tg3-3.43f]# pwd  
    
    /usr/local/src/tg3-343f/tg3-3.43f  
    
    [[root@localhost](mailto:root@localhost) tg3-3.43f]# make  
    
    make -C /lib/modules/2.6.9-34.EL/build  
    
    SUBDIRS=/usr/local/src/tg3-343f/tg3-3.43f modules  
    
    make[1]: Entering directory `/usr/src/kernels/2.6.9-34.EL-x86_64'  
    
    CC [M] /usr/local/src/tg3-343f/tg3-3.43f/tg3.o  
    
    Building modules, stage 2.  
    
    MODPOST  
    
    CC /usr/local/src/tg3-343f/tg3-3.43f/tg3.mod.o  
    
    LD [M] /usr/local/src/tg3-343f/tg3-3.43f/tg3.ko  
    
    make[1]: Leaving directory `/usr/src/kernels/2.6.9-34.EL-x86_64'  
    
    [[root@localhost](mailto:root@localhost) tg3-3.43f]#
    
    Now drop your module inside this dir tree, packit up:  
    
    #[[root@localhost](mailto:root@localhost) ~]# find 2.6.9-34.EL | cpio -ovF modules.cgz  
    
    cpio: 2.6.9-34.EL: truncating inode number  
    
    2.6.9-34.EL  
    
    cpio: 2.6.9-34.EL/x86_64: truncating inode number  
    
    2.6.9-34.EL/x86_64  
    
    cpio: 2.6.9-34.EL/x86_64/acenic.ko: truncating inode number  
    
    2.6.9-34.EL/x86_64/acenic.ko  
    
    cpio: 2.6.9-34.EL/x86_64/fat.ko: truncating inode number  
    
    2.6.9-34.EL/x86_64/fat.ko  
    
    cpio: 2.6.9-34.EL/x86_64/3w-9xxx.ko: truncating inode number  
    
    2.6.9-34.EL/x86_64/3w-9xxx.ko  
    
    cpio: 2.6.9-34.EL/x86_64/megaraid_mm.ko: truncating inode number  
    
    etc...
    
    Compress modules.cgz, rename it  
    
    [[root@localhost](mailto:root@localhost) ~]# gzip modules.cgz  
    
    [[root@localhost](mailto:root@localhost) ~]# gunzip modules.cgz.gz  
    
    [[root@localhost](mailto:root@localhost) ~]# ls -lh modules.cgz  
    
    -rw-r--r-- 1 root root 9.6M Mar 27 13:11 modules.cgz  
    
    [[root@localhost](mailto:root@localhost) ~]# gzip modules.cgz  
    
    ls [[root@localhost](mailto:root@localhost) ~]# ls -lh modules.cgz.*  
    
    -rw-r--r-- 1 root root 3.1M Mar 27 13:11 modules.cgz.gz  
    
    -rw-r--r-- 1 root root 8.9M Mar 27 12:43 modules.cgz.old  
    
    [[root@localhost](mailto:root@localhost) ~]# mv modules.cgz.gz modules.cgz
    
    or just can also run:  
    
    [[root@localhost](mailto:root@localhost) ~]# find 2.6.9-34.EL |cpio -H newc -ov |gzip -9 -c - &gt; ../modules.cgz
    
    Copy the modified mod file to the initrd tree:
    
    [[root@localhost](mailto:root@localhost) ~]# cp modules.cgz a/modules/  
    
    cp: overwrite `a/modules/modules.cgz'? y  
    
    [[root@localhost](mailto:root@localhost) ~]# sync  
    
    [[root@localhost](mailto:root@localhost) ~]#
    
    edit modules.pcimap  
    
    Append a line such as this:  
    
    tg3 0x000014e4 0x0000166a 0xffffffff 0xffffffff  
    
    0x00000000 0x00000000 0x0
    
    The first entry is the module, the second and third are obtained from  
    
    lspci -n command (it happened to be 14e4 166a in my case)
    
    **_For Kernel 2.4_**  
    
    umount the loopfilesys, your new initrd is ready!  
    
    recompress the initrd.img (and rename too) and your shiny new initrd  
    
    is ready for use!
    
    **_For Kernel 2.6_**  
    
    Pack the initrd.img file:  
    
    [[root@localhost](mailto:root@localhost) ~]# cd  
    
    [[root@localhost](mailto:root@localhost) ~]# find . |cpio -H newc -ov |gzip -9 -c - &gt; ../initrd.img
    
    **_Summary for Kernel 2.6 initrd's:_**  
    
    #mkdir /tmp/rhel5.3  
    
    #cd /tmp/rhel5.3  
    
    #zcat /install/rhels5.3/x86_64/images/pxeboot/initrd.img | cpio -dumi
    # ls -l
    total 36  
    
    lrwxrwxrwx 1 root 252 4 Apr 16 19:27 bin -&gt; sbin  
    
    drwxr-xr-x 2 root 252 4096 Oct 21 02:04 dev  
    
    drwxr-xr-x 3 root 252 4096 Apr 16 19:27 etc  
    
    lrwxrwxrwx 1 root 252 10 Apr 16 19:27 init -&gt; /sbin/init  
    
    drwxr-xr-x 2 root 252 4096 Apr 16 19:27 modules  
    
    drwxr-xr-x 2 root 252 4096 Oct 21 02:04 proc  
    
    drwxr-xr-x 2 root 252 4096 Apr 16 19:27 sbin  
    
    drwxr-xr-x 2 root 252 4096 Oct 21 02:04 selinux  
    
    drwxr-xr-x 2 root 252 4096 Oct 21 02:04 sys  
    
    drwxr-xr-x 2 root 252 4096 Oct 21 02:04 tmp  
    
    drwxr-xr-x 6 root 252 4096 Apr 16 19:27 var
    
    #cd modules/
    # ls -l
    total 4432  
    
    -rw-r--r-- 1 root 252 6017 Oct 21 02:04 module-info  
    
    -rw-r--r-- 1 root 252 120732 Oct 21 02:04 modules.alias  
    
    -rw-r--r-- 1 root 252 4298146 Apr 15 01:04 modules.cgz  
    
    -rw-r--r-- 1 root 252 14940 Oct 21 02:04 modules.dep  
    
    -rw-r--r-- 1 root 252 66292 Oct 21 02:04 pci.ids ls
    
    #mkdir m  
    
    #cd m  
    
    #zcat ../modules.cgz | cpio -dumi
    # ls -lR
    .:  
    
    total 4  
    
    drwx------ 3 root root 4096 Apr 16 19:28 2.6.18-120.el5
    
    ./2.6.18-120.el5:  
    
    total 4  
    
    drwx------ 2 root root 4096 Apr 16 19:28 x86_64
    
    ./2.6.18-120.el5/x86_64:  
    
    total 19392  
    
    -rwxr--r-- 1 root 252 56216 Oct 21 02:04 3c574_cs.ko  
    
    -rwxr--r-- 1 root 252 53128 Oct 21 02:04 3c589_cs.ko  
    
    -rwxr--r-- 1 root 252 110448 Oct 21 02:04 3c59x.ko  
    
    -rwxr--r-- 1 root 252 83472 Oct 21 02:04 3w-9xxx.ko  
    
    -rwxr--r-- 1 root 252 70064 Oct 21 02:04 3w-xxxx.ko  
    
    -rwxr--r-- 1 root 252 63824 Oct 21 02:04 8021q.ko  
    
    ...
    
    Add/Remove modules, When you finish , pack again the modules.cgz:
    
    For example, to remove qlogic modules from the initrd.img:  
    
    #rm -f 2.6.18-120.el5/x86_64/qla*
    
    #find . |cpio -H newc -ov |gzip -9 -c - &gt; ../modules.cgz  
    
    #cd ..  
    
    Remove the temporary m directory:  
    
    #rm -rf m
    
    Pack the initrd.img file:  
    
    #cd ..  
    
    #find . |cpio -H newc -ov |gzip -9 -c - &gt; /install/rhels5.3/x86_64/images/pxeboot/initrd.img
    
    We are done !!!  
    
    