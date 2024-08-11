#  Requirements for Building and Using the Kernel
### compile and build kernel
build a kernel: a compiler, a linker, and a make utility             
- The Linux kernel is written in the C programming language, with a small amount of assembly language in some places. To build the kernel, the gcc C compiler must be used [gcc](http://gcc.gnu.org) (```gcc --version```)          

- The Ccompiler, gcc, does not do all of the compiling on its own. It needs an addi tional set of tools known as binutils to do the linking and assembling of source files [buildint]( http://www.gnu.org/software/binutils) (To determine which version of binutils you have on your system, run the following command ```ld -v```)      

- The [make](http://www.gnu.org/software/make) is a tool that walks the kernel source tree to determine which files need to be compiled, and then calls the compiler and other build tools to do the work in building the kernel (```make --version```).          

### Some common packages
- The util-linux package is a collection of small utilities that do a wide range of different tasks. Most of these utilities handle the mounting and creation of disk partitions and manipulation of the hardware clock in the system [link](http://www.kernel.org/pub/linux/utils/util-linux) (To determine which version of the util-linux package you have on your system, run the following command: ```fdformat --version```)        

- The module-init-tools package is needed if you wish to use Linux kernel modules. A kernel module is a loadable chunk of code that can be added to or removed from the kernel while the kernel is running. It is useful to compile device drivers as modules and then load only the ones that correspond to the hardware present in the system [link](http://www.kernel.org/pub/linux/utils/kernel/module-init-tools) (To determine which version of the module-init-tools package you have on your system, run the following command: ```depmod -V```)         

- The Filesystem-Specific Tools  ext2/ext3/ext4 to work with any of these filesystems, you must have the e2fsprogs package. If you wish to download and install this package yourself, you can find it at [link](http://e2fsprogs.sourceforge.net) (To determine which version of e2fsprogs you have on your system, run the following command: ```tune2fs```)    

- To use the JFS filesystem from IBM, you must have the jfsutils pacakge. If you wish to download and install this package yourself, you can find it at [link](http://jfs.sourceforge.net) (To determine which version of jfsutils you have on your system, run the following command: ```fsck.jfs -V```)            

- To use the ReiserFS filesystem, you must have the reiserfsprogs package. If you wish to download and install this package yourself, you can find it at [link](http://www.namesys.com/download.html) (To determine which version of reiserfsprogs you have on your system, run the following command: ```reiserfsck -V```)            

- To use the XFS filesystem from SGI, you must have the xfsprogs package. If you wish to download and install this package yourself, you can find it at [link](http://oss.sgi.com/projects/xfs) (To determine which version of xfsprogs you have on your system, run the following command: ```xfs_db ```)           

- To use the quota functionality of the kernel, you must have the quota-tools package. You can find it at [link](http://sourceforge.net/projects/linuxquota) (To determine which version of quota-tools you have on your system, run the following command: ```quota -V```)            

- To use the NFS filesystem properly, you must have the nfs-utils package. You can find it at [link](http://nfs.sf.net) (To determine which version of nfs-utils you have on your system, run the following command: ```showmount --version```)          

Alternately have many other tools...         
Note: Some distributions have different names.        
Ex: Debian, call this package quota instead of quota-tools. Debian, call this package nfs-common instead of nfs-utils ...           


### udev
udev is a program that enables Linux to provide a persistent device-naming systemin the /dev directory. Almost all Linux distributions use udev to manage the /dev directory. You can find it at [link](http://www.kernel.org/pub/linux/utils/kernel/hotplug/udev.html) ( To determine which version of udev you have on your system, run the following command: ```udevinfo -V```)          

### Process tools
The package procps includes the commonly used tools ps and top, as well as many other handy tools for managing and monitoring processes running on the system. You can find it at [link](http://procps.sourceforge.net) (To determine which version of procps you have on your system, run the following command: ```ps --version```)           

# Retrieving the Kernel Source
### Where to Find the Kernel Source
http://www.kernel.org         
```
sudo apt-get udpate
mkdir linux-kernel
cd $_
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.10.3.tar.xz
tar -xf linux-6.10.3.tar.xz
```

# Configuring and Building
### config
The kernel configuration is kept in a file called .config in the top directory of the kernel source tree. If you have just expanded the kernel source code, there will be no .config file, so it needs to be created. It can be created from scratch, created by basing it on the “default configuration,” taken from a running kernel version, or taken from a distribution kernel release.          
```
sudo apt install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
cd linux-6.10.3.tar.xz
make config 
The four choices are:
 y : Build directly into the kernel.
 n : Leave entirely out of the kernel.
 m : Build as a module, to be loaded if needed.
 ? : Print a brief descriptive message and repeat the prompt.
(to specify config option. We have more than two thousand different configuration options so ```make defconfig``` is a good choice)
make menuconfig
make -j6(to rebuild the kernel)
```
To build only a specific subdirectory or a single file within the whole kernel tree.
```
#build the files in the drivers/usb/serial directory
make drivers/usb/serial

#build all the needed files in that directory and link the final module images
make M=drivers/usb/serial

#rebuild the kernel
make -j6

#only rebuild module
make drivers/usb/serial/visor.ko
```

###  Source in One Place, Output in Another
For example, if the kernel source is located on a CD-ROM mounted on /mnt/cdrom/ and you wish to place the built files in your local directory ~/home/guru/linux-kernel
```
cd /mnt/cdrom/$(uname -r)
make O=~/home/guru/linux-kernel
```
All the files build will create under ~/home/guru/linux-kernel directory. O= option should also be passed to the configuration options of the build so that the configuration is correctly placed in the output directory and not in the directory containing the source code.

### Different Architectures
The kernel build system allows you to specify a different architecture from the current system with the ARCH= argument
For example, to get the default kernel configuration of the x86_64 architecture, you would enter
```
make ARCH=x86_64 defconfig
```
To build the whole kernel with an ARM toolchain located in /usr/local/bin/, you would enter:
```
make ARCH=arm CROSS_COMPILE=/usr/local/bin/arm-linux-
```
Using the distcc or ccache programs, both of which help greatly reduce the time it takes to build a kernel. To use the ccache program as part of the build system, enter:
```
#with ccache
make CC="ccache gcc"
#both distcc and ccache
make CC="ccache distcc"
```
# Installing and Booting from a Kernel (auto)
This will install all the modules that you have built and place them in the proper location in the filesystemfor the new kernel to properly find. Modules are placed in the ```/lib/modules/kernel_version``` directory, where kernel_version is the kernel version of the new kernel you have just built.
If you have built any modules and want to use use this method to install a kernel, first enter:         
```
sudo make modules_install
```
After the modules have been successfully installed, the main kernel image must be installed
```
make install
```
This will kick off the following process:          
1. The kernel build systemwill verify that the kernel has been successfully built properly.           
2. The build systemwill install the static kernel portion into the /boot directory and name this executable file based on the kernel version of the built kernel.       
3. Any needed initial ramdisk images will be automatically created, using the modules that have just been installed during the modules_install phase.       
4. The bootloader programwill be properly notified that a new kernel is present, and it will be added to the appropriate menu so the user can select it the next time the machine is booted.       
5. After this is finished, the kernel is successfully installed, and you can safely reboot and try out your new kernel image. Note that this installation does not overwrite any older kernel images, so if there is a problem with your new kernel image, the old kernel can be selected at boot time.           

# Installing and Booting from a Kernel (by hand)
If your distribution does not have a installkernel command, or you wish to just do the work by hand to understand the steps involved, here they are:        
```
#The modules must be installed
sudo make modules_install
#The static kernel image must be copied into the /boot directory. For an i386-based kernel, do the following:
make kernelversion (output: 2.6.17.11 use this value in place of the text KERNEL_VERSION in the steps bellow)
cp arch/i386/boot/bzImage /boot/bzImage-KERNEL_VERSION
cp System.map /boot/System.map-KERNEL_VERSION
```
Scripts:
```
#!/bin/sh
 #
 # installs a kernel
 #
 make modules_install
 # find out what kernel version this is
 for TAG in VERSION PATCHLEVEL SUBLEVEL EXTRAVERSION ; do
    eval `sed -ne "/^$TAG/s/ //gp" Makefile`
 done
 SRC_RELEASE=$VERSION.$PATCHLEVEL.$SUBLEVEL$EXTRAVERSION
 # figure out the architecture
 ARCH=`grep "CONFIG_ARCH " include/linux/autoconf.h | cut -f 2 -d "\""`
 Installing and
 Booting
 # copy the kernel image
 cp arch/$ARCH/boot/bzImage /boot/bzImage-"$SRC_RELEASE"
 # copy the System.map file
 cp System.map /boot/System.map-"$SRC_RELEASE"
 echo "Installed $SRC_RELEASE for $ARCH"
```

# Modifying the Bootloader for the New Kernel
Common bootloader: U-Boot, GRUB, LILO
To determine which bootloader your system uses, look in the /boot/ directory.          
```
ls -F /boot | grep grub
ls -F /boot | grep lilo ( If this directory is not present, look for the presence of the /etc/lilo.conf)
ls -F /boot | grep u-boot
```
To let GRUB know that a new kernel is present, all you need to do is modify the /boot/grub/menu.lst file. For full details on the structure of this file, and all of the different options available, please see the GRUB info pages:        
```
info grub
```
To let GRUB know that a new kernel is present, all you need to do is modify the /boot/grub/menu.lst file add new version base on template. After modify reboot the system new kernel will apply.         

# Upgrading a Kernel 
This part show how easy it is to update a kernel from an older versions, while still retaining all of the configuration options from the previous one.               
First off, please back up the .config file in the kernel source directory. You have spent some time and effort into creating it, and it should be saved in case some thing goes wrong when trying to upgrade.           
```
cd linux-6.10.3
cp .config ../good_config
```
Only five simple steps are needed to upgrade a kernel from a previously built one:          
1. Get the new source code.         
2. Apply the changes to the old source tree to bring it up to the newer level.          
3. Reconfigure the kernel based on the previous kernel configuration.           
4. Build the new kernel.                
5. Install the new kernel.              
Assumsion: your kernel current is 2.6.17.9 and you need upgrade
A kernel patch file will upgrade the source code from only one specific release to another specific release. Here is how the different patch files can be applied:          
Stable kernel patch apply to the base kernel version.This means that the 2.6.17.10 patch will only apply to the 2.6.17 kernel release.The2.6.17.10 kernel patch will not apply to the 2.6.17.9 kernel or any other release.         
Base kernel release patch only apply to the previous base kernel version. This means that the 2.6.18 patch will only apply to the 2.6.17 kernel release. It will not apply to the last 2.6.17.y kernel release, or any other release.            
As we want to go from the 2.6.17.9 kernel release, to the 2.6.17.11 release, we will need to download two different patches. We will need a patch from the 2.6.17.9 release to the 2.6.17.10 release, and then from the 2.6.17.10 release to the 2.6.17.11 release.*       
### Applying the Patch
As the patches we have downloaded are compressed, the first thing to do is uncompress them with the bzip2 command:      
```
bzip2 -dv patch-2.6.17.9-10.bz2
bzip2 -dv patch-2.6.17.10-11.bz2
```
Now we need to apply the patch files to the kernel directory. Go into the directory:      
```
cd linux-2.6.17.9
#Now run the patch program to apply the first patch moving the source tree from the 2.6.17.9 to the 2.6.17.10
patch -p1 < ../patch-2.6.17.9-10
#Verify that the patch really did work properly and that there are no errors or warnings in the output ofthe patch program
head -n 5 Makefile (to see kernel version after apply patch EXTRAVERSION)
#apply patch 11
patch -p1 < ../patch-2.6.17.10-11
```
Nowthat the source code is successfully updated to the version you wish to use      
```
cd ..
#rename to new version
mv linux-2.6.17.9 linux-2.6.17.11
```

### Reconfigure the Kernel
Previously, we used the make menuconfig or gconfig or xconfig method to change different configuration options. But once you have a working configuration, the only thing that is necessary is to update it with any new options that have been added to the kernel since the last release. To do this, the ```make oldconfig``` and ```make silentoldconfig``` options should be used.     
```make oldconfig``` takes the current kernel configuration in the .config file, and updates it based on the new kernel release. To do this, it prints out all configuration questions, and provides an answer for them if the option is already handled in the configuration file. If there is a new option, the program stops and asks the user what the new configuration value should be set to. After answering the prompt, the program continues on until the whole kernel configuration is finished.

```make silentoldconfig``` works exactly the same way as oldconfig, but it does not print anything to the screen, unless it needs to ask a question about a new configura tion option.

```
cd linux-2.6.17.11
make silentoldconfig
```

# Customizing a Kernel
### Where Is the Kernel Configuration?
Almost all distributions provide the kernel configuration files as part of the distribution kernel package.Read the distribution-specific documentation for how to find these configurations. It is usually somewhere below the /usr/src/linux/ directory tree.         
If the kernel configuration is hard to find, look in the kernel itself.Most distribu tion kernels are built to include the configuration within the /proc filesystem.To determine if this is true for your running kernel, enter:            
```
ls /proc/config.gz (filename is present, copy this file to your kernel source direc tory and uncompress it and rebuild kernel)
```
### Finding Which Module Is Needed?
- A configuration file that comes from a distribution takes a very long time to build, because of all of the different drivers being built.You want to build only the drivers for the hardware that you have, which will save time on building the kernel, and allows you to build some or all of the drivers into the kernel itself, possibly saving a bit of memory, and on some architectures, making for a faster running system.To cut your drivers down, you need to determine which modules are needed to drive your hardware.We will walk though two examples of how to find out what driver is needed to control what piece of hardware.               
- Several locations on your system store useful information for determining which devices are bound to which drivers in a running kernel.The most important location is a virtual filesystem called sysfs. sysfs should always be mounted at the /sys location in your filesystem by the initialization scripts of your Linux distribution. sysfs provides a glimpse into how the different portions of the kernel are hooked together, with many different symlinks pointing all around the filesystem.
Ex: Determining the network driver        
```
#find out which PCI device is controlling network
ls /sys/class/net/ (output eth0  eth1  eth2  lo)
#get further info
/sbin/ifconfig -a (output present the eth0 device is the network device that is active and working)
#Now that we have determined that we want to make sure the eth0 device will be working in our new kernel, we need to find which driver is controlling it.This is simply a matter of walking #the different links in the sysfs filesystem, which can be done in a one-line command:
basename `readlink /sys/class/net/eth0/device/driver/module` (output e1000 module named e1000 is controlling the eth0 network device) 

cd ~/linux/linux-2.6.17.8
find -type f -name Makefile | xargs grep e1000

#Now you have the information you need to configure the kernel.Run the menu configuration tool:
make menuconfig (Then press the / key and type in the configuration option E1000 to search. The kernel configuration system will then tell you exactly where to select the option to enable this module)
```
Ex: A USB device
```
ls /sys/class/tty/ | grep USB
#find the controlling module
basename `readlink /sys/class/tty/ttyUSB0/device/driver/module` (output pl2303)
#Then search the kernel source tree to find the configuration option that you need to enable
cd ~/linux/linux-2.6.17.8
find -type f -name Makefile | xargs grep pl2303
make menuconfig (/ key find keyword CONFIG_USB_SERIAL_PL2303)
```
Scripts: This script goes through sysfs and finds all files called modalias.The modalias file contains the module alias that tells the modprobe command which module should be loaded to control this device.
```
#!/bin/bash
#
# find_all_modules.sh
#
for i in `find /sys/ -name modalias -exec cat {} \;`; do
   /sbin/modprobe --config /dev/null --show-depends $i ;
done | rev | cut -f 1 -d '/' | rev | sort -u
```
### PCI Devices
A PCI (Peripheral Component Interconnect) device is any hardware component that connects to a computer's motherboard through the PCI bus, an industry-standard bus for attaching peripheral devices to a computer's central processing unit (CPU)           
To get a list of all PCI devices:         
```
which lspci
/usr/sbin/lspci

cd /sys/bus/pci/devices/
ls
cd 0000:06:04.0
cat vendor (output 0x10ec)
cat device (output 0x8139)
cd ~/linux/linux-2.6.17.8/
grep -i 0x10ec include/linux/pci_ids.h (output PCI_VENDOR_ID_REALTEK is what will probably be used in any kernel driver that purports to support devices from this manufacturer)
grep -i 0x8139 include/linux/pci_ids.h
#Now look for driver source files referring to this vendor definition:
grep -Rl PCI_VENDOR_ID_REALTEK *
```
In summary, here are the steps needed in order to find which PCI driver can control a specific PCI device:
1. Find the PCI bus ID of the device for which you want to find the driver, using lspci.
2. Go into the /sys/bus/pci/devices/0000:bus_id directory, where bus_id is the PCI bus ID found in the previous step.
3. Read the values of the vendor and device files in the PCI device directory.
4. Move back to the kernel source tree and look in include/linux/pci_ids.h for the PCI vendor and device IDs found in the previous step.
5. Search the kernel source tree for references to those values in drivers.Both the vendor and device ID should be in a struct pci_device_id definition.
6. Search the kernel Makefiles for the CONFIG_ rule that builds this driver by using find and grep:
```
find -type f -name Makefile | xargs grep DRIVER_NAME
```
7. Search in the kernel configuration system for that configuration value and go to the location in the menu that it specifies to enable that driver to be built.

### Root Filesystem
The root filesystem is the filesystem from which the main portion of the running system boots. It contains all of the initial programs that start up the distro, and also usually contains the entire system configuration for the machine.
```
mount | grep " / "
```

# Resize virtualbox machine 
```
VBoxManage modifyhd YOUR_HARD_DISK.vdi --resize SIZE_IN_MB
```