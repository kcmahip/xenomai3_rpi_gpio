[TOC]


install xenomai 3.0.8 on rpi3B+
===============================

Note that this image also works on all other rpi3's and rpi2's. (Cortex-A7/ARMv7 and Cortex-A53/ARMv8 cpu's are compatible)
But not on the rpi1 and rpi zero because they have a different type of ARM processor. (ARM1176JZF-S/ARMv6Z)
However using a different default config for the kernel build we can also easily build an image for rpi1 and rpi zero.
We can then put both images on the sd-card and on boot the bootloader will automatically load the right kernel 
for the specific raspbian hardware board. For more details see: 

    https://www.raspberrypi.org/documentation/linux/kernel/building.md => describes default configs to use in bulid
    https://pi-ltsp.net/advanced/kernels.html
    https://en.wikipedia.org/wiki/Raspberry_Pi#Specifications
     and 
    install_xenomai-3-3.0.5_on_rpi1__ipipe_git.txt 


idea: just upgrade old image
----------------------------    
             
idea: 
     my old xenomai image was build for debian jessie  for the rpi3b, but not for the rpi3b+ which has newer hardware
     
     so just upgrade only the kernel and xenomai in our  debian jessie installation for xenomai 3.05 which
     to also let is support rpi3b+ 
     
answer:
 
  => cannot do, need new stretch debian release because jessie doesn't support the new hardware
     also the device tree for the rpi3b+ was not available then yet ! (hardware of 3b+ was made later!)
       
       
       src:   https://www.raspberrypi.org/forums/viewtopic.php?t=223806

          The firmware in Jessie was released before the 3B+ even existed. 
          It does not know how to boot it up properly. So you need the latest firmware.

          The 3B+ also has different peripherals (USB, Ethernet, Wifi...) so it also 
          needs new kernel drivers that  are not present in Jessie. Jessie is no longer supported.
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                  `-> new hardware only suppored in stretch, not in jessie!
                  
          You are probably much better to address any issues you have with Stretch, rather than trying to get Jessie working on a 3B+.


          => it also needs new kernel drivers,  so upgrading my old system to new system, with old kernel and its modules 
             won't work because misses the new hardware drivers
   
          => so solution: recompile old kernel on new system won't work because lack of new device driver code!




   
versions
========

So we need to upgrade to debian stretch to support rpi3b+


First choose xenomai/ipipe version to patch xenomai on raspbian kernel  

    https://www.xenomai.org/downloads/ipipe/v4.x/arm/
  
    [TXT] ipipe-core-4.14.110-arm-7.patch 2019-04-03 19:24  595K   ==> to recent patches are applied -> can contain still a bug
    [TXT] ipipe-core-4.14.36-arm-1.patch  2018-06-12 19:12  587K  
    [TXT] ipipe-core-4.14.62-arm-2.patch  2018-09-13 14:57  591K  
    [TXT] ipipe-core-4.14.71-arm-3.patch  2018-09-24 15:04  594K  
    [TXT] ipipe-core-4.14.71-arm-4.patch  2018-10-13 15:59  594K  
    [TXT] ipipe-core-4.14.85-arm-5.patch  2018-12-06 12:40  594K  
    [TXT] ipipe-core-4.14.85-arm-6.patch  2018-12-19 20:44  594K  
    [TXT] ipipe-core-4.19.33-arm-1.patch  2019-04-04 10:53  593K   
    [TXT] ipipe-core-4.4.71-arm-9.patch   2017-10-03 12:35  672K  
    [TXT] ipipe-core-4.9.51-arm-4.patch   2018-03-26 09:16  700K   ==> seems to be stable!!!! 
   
   
     => choose ipipe version which seems the stables =
         -  take   ipipe-core-4.9.51-arm-4.patch
         
then match on that the raspbian version 
  for raspbian versions look at https://github.com/raspberrypi/linux
   => gives raspbian rpi 4.9.y


we use the latest xenomai version:   xenomai 3.0.8   (april 2019)   


on the sd card install a lite raspbian version so that image becomes small
    https://www.raspberrypi.org/downloads/raspbian/


    Raspbian Stretch Lite
    Minimal image based on Debian Stretch
    Version: April 2019
    Release date: 2019-04-08
    Kernel version: 4.14
     => 2019-04-08-raspbian-stretch-lite.zip
    

installation ingredients
========================

  based on  
    install_xenomai-3-3.0.5_on_rpi2__using_official_ipipe_patch__external_bcm_patch__pinctrl-bcm2835_patchfrom_ipipe_repos.txt
 
  documentation how to build kernel for raspbian:
       https://www.raspberrypi.org/documentation/linux/kernel/building.md
        => describes how to install cross compiler on linux
        => describes how to compile kernel:
            For Pi 1, Pi Zero, Pi Zero W, or Compute Module:

                cd linux
                KERNEL=kernel
                make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcmrpi_defconfig
            
            For Pi 2, Pi 3, Pi 3+, or Compute Module 3:

                cd linux
                KERNEL=kernel7
                make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig

            Then, for both:

                make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs

         => describes howto  copy the kernel/modules and Device Tree blobs onto the SD card

   
 
  Note in   xenomai308_image_virtualbox.txt we installed the vanilla(kernel.org) kernel 4.9.144  with
  '4.9.144-3.1-debian' debian specific kernel patches  applied  to the vanilla kernel.
     
  However for the rpi  we use raspbian kernel source!!
  
  We use raspbian kernel and not the vanilla kernel from kernel.org because that kernel has better gpio and device_tree
  support then the vanilla kernel for the raspberry pi. =>  see vanilla_kernel_or_raspbian_kernel.txt    

  If we clone rasbian kernel source for kernel 4.9 and
  we run  'make kernelversion' it gives the full version 4.9.80
 
  However the latest ipipe for the 4.9 kernel we have is 
     https://www.xenomai.org/downloads/ipipe/v4.x/arm/ipipe-core-4.9.51-arm-4.patch
  so kernel versions do not entirely match  : 4.9.80 vs 4.9.51 
  Also the ipipe patch is for the vanilla(kernel.org) kernel and not for the rpi-4.9 kernel.
  The rpi-4.9 kernel is based on the vanilla kernel, but has extra specific changes for the raspberry pi.
     
  However probably the patch will just work, we just try it!   
 
 
  Compared to installation describe in 

   install_xenomai-3-3.0.5_on_rpi2__using_official_ipipe_patch__external_bcm_patch__pinctrl-bcm2835_patchfrom_ipipe_repos.txt

  We have NEW ingredients: 
 
     linux distr. with kernel 4.9 xenomai-3.0.8
 
  so ingredients are
   a) xenomai-3.0.8
   b) raspbian Raspbian Stretch Lite - with kernel 4.9.y   (full version 4.9.80)           \ ,-> all 4.9 !
   c) ipipe-core-4.9.51-arm-4.patch                        (for  version 4.9.51)           /
   d) pinctrl-bcm2835.c  from rpi specific branch 'vendors/raspberry/ipipe-4.1'      => for 4.1 \,-> still works??
   e) patch-xenomai-3-on-bcm-2709.patch   extra raspberry pi specific patch -> worked on 4.1    /     


  ad e) patch-xenomai-3-on-bcm-2709.patch is not needed anymore because the specific kernel source code
        the patch is applied on is removed from the rpi-4.9 kernel (also for the 4.9 vanilla kernel)
        note: before 4.8 the linux kernel used to have two different drivers for the gpio chip,
              however since the 4.8 kernel we only have a single driver "pinctrl-bcm2835"  
              with config PINCTRL_BCM2835 for all raspbian hardware boards!!
              => src: ../gpio/raspberry_pi__gpio_chip__two_drivers.txt 
       
  ad d)
     We have a xenomai/ipipe patched pinctrl-bcm2835.c  from xenomai/ipipe's rpi specific branch 'vendors/raspberry/ipipe-4.1'    
     which you can download at https://gitlab.denx.de/Xenomai/ipipe/raw/vendors/raspberry/ipipe-4.1/drivers/pinctrl/bcm/pinctrl-bcm2835.c
     However this was for the rpi-4.1 kernel and didn't work anymore for the rpi-4.9 kernel.
     The  ipipe-core-4.9.51-arm-4.patch  has some new patches for the vanilla 4.9 kernel, but this misses the raspbian
     specific enhancements.
     So with some work I derived a new ipipe/xenomai patch file for pinctrl-bcm2835.c in the rpi-4.9 kernel source.
     This patch 'pinctrl-bcm2835.c.rpi-4.9.patch' must be applied to  pinctrl-bcm2835.c in the rpi-4.9 kernel source
     and we should remove the  ipipe/xenomai patches for pinctrl-bcm2835.c  from the ipipe-core-4.9.51-arm-4.patch.
     
     For details how derived this patch see:  
     
     
        how_pinctrl-bcm2835_patch_for_rpi-4.9_is_derived/__README__.txt
        

  ad c) we need to patch the ipipe-core-4.9.51-arm-4.patch a little so that it also worked
       on the raspbian 4.9.80 kernel. We also removed all patches for 'pinctrl-bcm2835.c' because
       we already have a specific patch for that. So we get finally a new patch:
       
       ipipe-core-4.9.51-arm-4.fixedforrasbian-4.9.80.patch
     
                
  so new ingredients are
   a) xenomai-3.0.8
   b) raspbian Raspbian Stretch Lite - with kernel 4.9.y   (full version rpi-4.9.80)          
   c) ipipe-core-4.9.51-arm-4.fixedforrasbian-4.9.80.patch (originally for  version 4.9.51, made compatible for rpi-4.9.80)           
   d) pinctrl-bcm2835.c.rpi-4.9.patch
     


installation recipe
====================
 

# 0. prepare 
# ------------

export BUILDDIR=/var/tmp/rpi3bplus
# build kernel with NUMCORES
export NUMCORES=4


mkdir $BUILDDIR   # place where download and build everything  
mkdir  $BUILDDIR/dist  # for ready to install files

# 1. get sources 
#---------------
#
# 1a. get raspbian linux 4.9.y
#
cd $BUILDDIR
git clone https://github.com/raspberrypi/linux.git
cd  linux/
git checkout  rpi-4.9.y

# just verify kernelversion with a make command:
make kernelversion
# which gives 4.9.80 which is full version -> number also used for directory for in /lib/modules/

cd ..
#
# 1b. get xenomai source (effects only linux kernel)
wget https://xenomai.org/downloads/xenomai/stable/xenomai-3.0.8.tar.bz2
tar -xjvf xenomai-3.0.8.tar.bz2
#
#
# 1c. get ipipe  patch  (effects both kernel and some drivers(using interrupts))
wget https://www.xenomai.org/downloads/ipipe/v4.x/arm/ipipe-core-4.9.51-arm-4.patch

#
# 1d. we derived a new patch for pinctrl-bcm2835.c for raspbian 4.9 kernel : pinctrl-bcm2835.c.rpi-4.9.patch
#     In the file  how_pinctrl-bcm2835_patch_is_derived/__README__.txt we describe how we derived this file. 
wget https://raw.githubusercontent.com/harcokuppens/xenomai3_rpi_gpio/master/install/how_pinctrl-bcm2835_patch_for_rpi-4.9_is_derived/pinctrl-bcm2835.c.rpi-4.9.patch





# 2.  apply kernel patches ( xenomai+ipipe, bcm( arch/arm/mach-bcm2709/.. and linux/drivers/pinctrl/bcm/pinctrl-bcm2835.c   )
# ------------------------
#
# 2a. apply xenomai (and ipipe) patch to kernel
# 
# first check if the ipipe patch applies well: (because little bit different kernel)
cd linux
patch  --dry-run  -p1  <  ../ipipe-core-4.9.51-arm-4.patch
# The argument --dry-run of the above execution allowed to verify if the patch is applied correctly without making changes.
# http://superuser.com/questions/755355/hunk-1-message-after-apply-a-patch
#  If the file to be patched has changed slightly since the patch was created, but the specific section 
# remained the same, patch can detect that and apply the patch appropriately.
# A hunk  message  means the file was successfully patched, but the first section that was patched was X lines earlier/later
# than was originally specified.
#

echo $?   # if it doesn't gives 0 as answer then dry-run of patch failed!
# in our case we got 1, so we need to fix the patch!!  => look at FAILED messages in patch output!!
# so we manually patched the patch file which gives us the new patch file:
#   ../ipipe-core-4.9.51-arm-4.fixedforrasbian-4.9.80.patch
# you can download this fixed patch with the command :

wget https://raw.githubusercontent.com/harcokuppens/xenomai3_rpi_gpio/master/install/how_pinctrl-bcm2835_patch_for_rpi-4.9_is_derived/ipipe-core-4.9.51-arm-4.fixedforrasbian-4.9.80.patch

# note: in this fixed patch we removed all patches for drivers/pinctrl/bcm/pinctrl-bcm2835.c because 
#       as already described above the raspbian patch is better for this file then the ipipe-4.9.y version.


# so dry running the ../ipipe-core-4.9.51-arm-4.fixedforrasbian-4.9.80.patch  patch succeeds ($?==0)
# thus we can apply the patch:


cd  ..
# finally really apply both xenomai and ipipe patch to kernel


IMPORTANT:
   edit xenomai-3.0.8/scripts/prepare-kernel.sh 
   and change the 'ln -sf' command into a 'cp' command
   => instead of soft linking the xenomai files in the linux src tree
      we now copy them!!  
   => this is needed for 'make deb-pkg' in the linux kernel source tree
      which otherwise makes packages with broken links!! (see linux headers below)


xenomai-3.0.8/scripts/prepare-kernel.sh  --linux=linux/  --arch=arm  --ipipe=ipipe-core-4.9.51-arm-4.fixedforrasbian-4.9.80.patch
# `-> xenomai script which does patch               `-> kernel source where patch applied     `-> ipipe patch file for kernel source



 
# 2b. apply a patch for pinctrl-bcm2835.c  :  pinctrl-bcm2835.c.patch  (see above how we get this)
       
       $ cd linux/drivers/pinctrl/bcm/
       $ patch   -p1  <  ../../../../pinctrl-bcm2835.c.rpi-4.9.patch
       patching file pinctrl-bcm2835.c
       Hunk #3 succeeded at 412 (offset -3 lines).
       Hunk #4 succeeded at 440 (offset -3 lines).
       Hunk #5 succeeded at 496 (offset -3 lines).
       Hunk #6 succeeded at 511 (offset -3 lines).
       Hunk #7 succeeded at 619 (offset -3 lines).
       Hunk #8 succeeded at 631 with fuzz 2 (offset -3 lines).
       Hunk #9 succeeded at 644 (offset -3 lines).
       Hunk #10 succeeded at 685 (offset -3 lines).
       Hunk #11 succeeded at 1097 (offset 7 lines).
       Hunk #12 succeeded at 1115 (offset 7 lines).
       Hunk #13 succeeded at 1127 with fuzz 2 (offset 7 lines).
       

       => patch successfull 

       cd ../../../..
       
       
# 3. configure linux kernel/module option
# ---------------------------------------


cd linux

# create default config for bcm2709
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig
# a default configuration for bcm2709 written to .config
# specialize config for xenomai
# install package needed for menuconfig :  apt-get  install libncurses5-dev
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

    CPU Power Management  --->
      CPU Frequency scaling  --->
        [ ] CPU Frequency scaling
      CPU idle  --->
        [ ] CPU idle PM support
    Kernel Features  --->
        [ ] Contiguous Memory Allocator
        [ ] Allow for memory compaction
    Kernel Hacking  --->
        [ ] KGDB: kernel debugger
    Boot options  --->
        Kernel command line type --->
            [X] Extend bootloader kernel arguments


    Xenomai Cobalt --->
         Drivers  --->
                 Real-time GPIO drivers  --->
                    <M> GPIO controller                          => with " [*] GPIO controller  " instead it builds directly in kernel, then no need to load module anymore!
                    [*]   Support for BCM2835 GPIOs                 `-> however we stick to default and just build it as module and load it at boot in /etc/rc.local
                    [ ]   Enable GPIO core debugging features


    note: in above  [ ]  means disable,  [X]  means enable



cd ..
        
      
      
# 4. build kernel and modules in linux dir      => cross compile
# ----------------------------------------
 
cd linux/ 
# build
time make -j $NUMCORES ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs
# -> real    6m45.452s

    real    12m8.649s
    user    28m1.124s
    sys     1m45.176s


# install in ../dist/ dir 
time make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=../dist modules_install

cd ..
      
      
 
 # EXTRA: install linux headers with Makefiles to build kernel modules on the pi itself    => cross compile
 #--------------------------------------------------------------------------------------
 #        note: 
 #          'make headers_install' installs only the headers, the Makefiles needed to build a kernel module are not there
 #          however with 'make deb-pkg' next to binary kernel packages it builds also a  linux-headers package
 #          which contains both headers and makefiles needed to build kernel modules. So we build this package 
 #          and copy it to the pi, and install it as a debian package on the pi

 cd linux/
 export NUMCORES=4 
 export KBUILD_DEBARCH=armhf 
 make ARCH=arm -j $NUMCORES  CROSS_COMPILE=arm-linux-gnueabihf-  deb-pkg

 # result: dpkg-deb: building package `linux-firmware-image-4.9.80-v7+' in `../linux-firmware-image-4.9.80-v7+_4.9.80-v7+-2_armhf.deb'.
 

 # copy the file  'linux-headers-4.9.80-v7+_4.9.80-v7+-2_armhf.deb'  to the pi 
 # then on the pi run :

    dpkg -i linux-headers-4.9.80-v7+_4.9.80-v7+-2_armhf.deb   

    # it seems that the scripts in the linux headers itself are not cross-compiled to arm, (intel instead)
    # so to use them to build kernel modules we first need to compile them as arm binaries, to
    # do this we can call 'make modules_prepare' which calls 'make scripts' . 
    cd /usr/src/linux-headers-4.9.80-v7+/
    # run following make command, which can give some errors, which should be ignored be giving it the -i option
    make -i modules_prepare   
    # building kernel modules should work fine now!  

 cd ..

 
 
 
 

# 5. built Xenomai user space part  ( kernel part is built with kernel above)      => cross compile
$ ------------------------------

## you need to have installed autoconf and libtool :
# sudo apt-get install autoconf
# sudo apt-get install libtool


# configure details see : https://xenomai.org/installing-xenomai-3-x/#_configuring

cd xenomai-3.0.8
./scripts/bootstrap 
./configure CFLAGS="-march=armv7-a  -mfloat-abi=hard -mfpu=neon -ffast-math" --host=arm-linux-gnueabihf --enable-smp --with-core=cobalt 

#
# note : http://xenomai.org/installing-xenomai-3-x/
#          –enable-smp
#              Turns on SMP support for Xenomai libraries.
#
#              Caution
#                  SMP support must be enabled in Xenomai libraries when the client
#                  applications are running over a SMP-capable kernel.

mkdir ../dist/xenomai
export DESTDIR=`realpath ../dist/xenomai`  # realpath because must be absolute path
make  DESTDIR="$DESTDIR"  install

 
 
 
 
6. copy build stuff in dist/ to raspbian sdcard 
-----------------------------------------------

# 0. install raspbian on sd card;  os x instructions : 


      note: diskutil is os x program!!  => do on mac!!

        diskutil list
            identify the disk (not partition) of your SD card. e.g. disk4 (not disk4s1)
        diskutil unmountDisk /dev/<disk# from diskutil>
            e.g. diskutil unmountDisk /dev/disk4
        sudo dd bs=1m if=<your image file>.img of=/dev/<disk# from diskutil>
            e.g. sudo dd bs=1m if=2014-09-09-wheezy-raspbian.img of=/dev/disk4
            (This will take a few minutes)
            
           

# 1. mount sdcard
#  become root otherwise you cannot access sdcard

sudo su -  

export BUILDDIR=/var/tmp/rpi3bplus
export SDCARD=/dev/sdb
export MOUNTPOINT=/media/sdcard

mkdir ${MOUNTPOINT}
mount ${SDCARD}2 ${MOUNTPOINT}
mount ${SDCARD}1 ${MOUNTPOINT}/boot


cd $BUILDDIR

# 2. copy Xenomai user space files to sd card
#  
cd dist/xenomai
chown -R root:root *
cp -a * ${MOUNTPOINT}/
cd ../..


# 3. copy kernel and modules 
#
# 3a. cp kernel and device tree files from linux/ dir  to /boot/ directory on image
cd linux/
cp arch/arm/boot/zImage ${MOUNTPOINT}/boot/
cp arch/arm/boot/dts/bcm27*.dtb ${MOUNTPOINT}/boot/
rm -rf ${MOUNTPOINT}/boot/overlays/*
cp arch/arm/boot/dts/overlays/*.dtb* ${MOUNTPOINT}/boot/overlays/

# 3b. cp modules from dist/lib/modules to sd card
cd ../
cp -r dist/lib/modules/* ${MOUNTPOINT}/lib/modules




# if you just rename the just installed file '/boot/zImage' to '/boot/kernel7.img'  then you don't need to edit
#  /boot/config.txt because the bootscript on the raspberry pi finds out it is running on pi2 or pi3 and the
#  automatically boots '/boot/kernel7' .    Note: on pi0 or pi1 it boots '/boot/kernel' instead!


mkdir -p ${MOUNTPOINT}/boot/orig_kernels
mv ${MOUNTPOINT}/boot/kernel* ${MOUNTPOINT}/boot/orig_kernels/
mv  ${MOUNTPOINT}/boot/zImage ${MOUNTPOINT}/boot/kernel7.img

# note: instead of renaming kernel7.img you can also specify manually in /boot/config.txt what kernel 
#       image you want to boot:
#
#          # update config for new kernel and device tree
#          # at end of /boot/config.txt add :
#
#          kernel=zImage
#          # for pi2
#          device_tree=bcm2709-rpi-2-b.dtb
#          # for pi3
#          #device_tree=bcm2710-rpi-3-b.dtb

umount ${MOUNTPOINT}/boot ${MOUNTPOINT}/



# 4. test installation

see test_xenomai_installation_and_test_gpio_pins_are_working_correct.txt




customize image
================
we configure so that we can easily
  * ssh login as root from another network host  => easy login and make/run programs
  * samba mount /root dir  => easy access in host to files with local mount so that we can use the editor of choice 
                              in the host to edit the files

you can log in with the following credentials:

      user:            pi
      passwd:          raspberry 

      user:            root
      passwd:          pi
      


configure pi
--------------

raspi-config   
 => goto localisation options
     => setup timezone to europe/amsterdam
     => setup keyboard to  us-keyboard

# login as user: pi
# become root user
sudo su -
# change password to 'pi'
passwd
# type twice 'pi'



            
utilities
---------

apt-get update

mouse support in console:
    apt-get install gpm 
gpio tool/libraries:
    apt-get install wiringpi
editors: 
    apt-get install vim
    apt-get install neovim   # because vim has no clipboard support builtin so copy pasting mouse doesn't work with vim, but does with neovim 
    apt-get install nano
    apt-get install joe  
for time sync: rdate -s time-srv.science.ru.nl    
    apt-get install rdate  
version control:
    sudo apt-get install git 
    sudo apt-get install subversion    
   
ssh server
------------

sudo systemctl enable ssh.service
sudo systemctl start ssh.service

edit  /etc/ssh/sshd_config so that :

    UsePAM yes
    PasswordAuthentication yes
    PermitRootLogin yes
    

/etc/init.d/ssh restart


# remove annoying warning:

rm /etc/profile.d/sshpwd.sh
 
 

samba server root homedir mount without passwd
----------------------------------------------

apt-get install samba

smbpasswd -a root    # note: use 'pi' as password
# edit  /etc/samba/smb.conf
# -> at homes section : set  'read only' to 'no'

/etc/init.d/samba restart


remove authentication from virtual consoles  => ctrl-alt-F1..12  ()
-------------------------------------------

in /lib/systemd/system/getty@.service
    ...
    ExecStart=-/sbin/agetty   --noclear %I $TERM
    ...
add "--autologin root" option :
    ...
    ExecStart=-/sbin/agetty  --autologin root --noclear %I $TERM        
    ...
    

fix kernel, so not by accident updated :            
-----------------------------------------
 # src: /home/harcok/doc/help/systeembeheer/linux/install/howto_install_and_or_update/apt.txt


     $ apt-mark hold raspberrypi-kernel
     $ apt-mark hold raspberrypi-bootloader   
   
 verify with command :
   
     $ apt-mark showhold
     raspberrypi-bootloader
     raspberrypi-kernel               

add custom Xenomai startup and login scripts 
---------------------------------------------
  see instructions at xenomai_startup/__README__xenomai_rpi_startup_config.txt


configure wlan
---------------

 => use raspi-config to setup wlan network
    note: country code must be set for rpi3b+ !!
 
 => also see notes/raspberrypi/networking/wifi_pi3.txt
 
 

SIDENOTES
=========


sidenote: compared to previous xenomai 3.05 build for rpi (4.1 kernel)
          NO patching of drivers/xenomai/gpio/Kconfig needed: 
          
      for 4.1 we had to fix xenomai Kconfig for xenomai gpio   =>  in xenomai source!!   => report to xenomai!
        
        in drivers/xenomai/gpio/Kconfig

            config XENO_DRIVERS_GPIO_BCM2835
                depends on MACH_BCM2708             => for pi1 , pi2 (and p3) has  MACH_BCM2709
        
             || change into
             \/
 
           config XENO_DRIVERS_GPIO_BCM2835      
             depends on MACH_BCM2708 || MACH_BCM2709    => xenomai source then applies for both pi1 and p2/p3



     for 4.9 we have :

            config XENO_DRIVERS_GPIO_BCM2835
                depends on MACH_BCM2708 || ARCH_BCM2835
                tristate "Support for BCM2835 GPIOs"
                help

                Enables support for the GPIO controller available from
                Broadcom's BCM2835 SoC.



         => ARCH_BCM2835 required  => ok, because ARCH_BCM2835 replaced MACH_BCM2708 MACH_BCM2709


                 see: notes/raspberrypi/raspberry_pi_BCM2708_or_BCM2835.txt 
                      which states MACH_BCM2708 is old and removed code for bcm chip in favor of new kernel.org compatible ARCH_BCM2835
                              
                        
 
         => so NO patching of drivers/xenomai/gpio/Kconfig needed!



# sidenote: 
 
        which defconfig to use??

              also see:  ../notes/raspberrypi/raspberry_pi_BCM2708_or_BCM2835.txt

                    $ ls ./arch/arm/configs/bcm*
                    ./arch/arm/configs/bcm2709_defconfig  ./arch/arm/configs/bcm2835_defconfig  ./arch/arm/configs/bcmrpi_defconfig

                    details:
             
                     ./arch/arm/configs/bcm2835_defconfig  -> default config added by kernel.org             
                     ./arch/arm/configs/bcmrpi_defconfig   -> added by raspbian  for rpi1        => used in my rpi1 build 
                     ./arch/arm/configs/bcm2709_defconfig  -> added by raspbian  for rpi2/3     => used in my rpi2/3 build   
             

                    https://www.raspberrypi.org/documentation/linux/kernel/configuring.md


                    For all models of Raspberry Pi 1 (includes Compute Module and Pi Zero):

                        $ KERNEL=kernel
                        $ make bcmrpi_defconfig

                    If you're cross-compiling, the second line should be:

                        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcmrpi_defconfig

                    For all models of Raspberry Pi 2/3 (includes Pi 3+ and Compute Module 3):

                        $ KERNEL=kernel7
                        $ make bcm2709_defconfig

                    If you're cross-compiling, the second line should be:

                        make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig



                    https://www.raspberrypi.org/forums/viewtopic.php?t=189631

                      On the page https://www.raspberrypi.org/documentati ... uilding.md explaining how to build the RPi kernel, there's
                      still mentioned bcm2709_defconfig but I was wondering if we should not start using bcm2835_defconfig since the latest
                      kernels put BCM2835 as hardware name with revision a01041 when running cat /proc/cpuinfo.
  
                        bcm2835_defconfig is the upstream defconfig.   => original authors 
  
                        bcm2709_defconfig is the one supported by Pi Towers.  => specific for rasperry pi ( pi towers)
  
                      => so use : bcm2709_defconfig   (note: there is no bcm2710_defconfig)

                      => note: also see vanilla_kernel_or_raspbian_kernel.txt that for vanilla kernel we don't have this 
                               config file, because it is special for the raspbian kernel!
                       
           official documentation: 
             https://www.raspberrypi.org/documentation/linux/kernel/building.md
               => says to use bcmrpi_defconfig for rpi1 and  bcm2709_defconfig  for rpi2 and rpi3        