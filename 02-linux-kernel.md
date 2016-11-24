##2. How to compile and install the Xilinx kernel in the MicroZedboard

###Prerequisites

- Linux PC or Virtual Machine with armhf toolchain installed
- 00-sd-card-setup: (Optional) Linux working on the microZed
- 01-xilinx-kernel-sources: Linux kernel sources from Xilinx

###Setup the environment for cross compiling

```sh
export CC=arm-linux-gnueabihf- 
export CROSS_COMPILE=${CC}
export LOADADDR=0x10008000
export ARCH=arm
```

Or use the script: kernel_toolchain_env.sh
	
Check that gcc is found and the architecture is ARM:
```sh
~$ ${CC}gcc --version
```

We are now ready to compile the kernel.

The kernel developers provide a Makefiles.
We can run all required tasks via "make".

###Configuring Linux

Create the default linux configuration for the Zynq (.config)
 ```sh
~/linux-xlnx$ make xilinx_zynq_defconfig
```
		
Configure the kernel (working based on the default config)
```sh
~/linux-xlnx$ make menuconfig
```

###Compiling the kernel

We build kernel image, modules separately. Modules can be installed via make.
Adapt the core count of your processor in this case 2.

```sh
~/linux-xlnx$ make menuconfig
~/linux-xlnx$ make uImage -j2
~/linux-xlnx$ make modules -j2
~/linux-xlnx$ make modules_install INSTALL_MOD_PATH=./my_modules 
```

###Device tree

 Use the device tree in arch/arm/boot/dts/zynq-zed.dts
 
 ```sh
~$ vim arch/arm/boot/dts/zynq-zed.dts
```

Change the line "chosen" parameters as next:
There is also a minor mistake in the include line we need to fix:

 ```diff
 /dts-v1/;
-#include "zynq-7000.dtsi"
+/include/ "zynq-7000.dtsi"
 
 / {
        model = "Zynq Zed Development Board";
@@ -31,8 +31,8 @@
        };
 
        chosen {
-               bootargs = "";
-               stdout-path = "serial0:115200n8";
+              bootargs = "console=ttyPS0,115200 root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlyprintk";
+              stdout-path = "/amba/serial@e0001000";
        };
 ```

 The result should be:
 
     	chosen {
     		bootargs = "console=ttyPS0,115200 root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait earlyprintk";
     		linux,stdout-path = "/amba/serial@e0001000";
     	};

Compile the source into a device tree binary (dtb):

 ```sh
~$ dtc -I dts -O dtb -o devicetree.dtb arch/arm/boot/dts/zynq-zed.dts 
 ```

To add or change existing peripherals edit this file:

vim arch/arm/boot/dts/zynq-7000.dtsi
	        

##(Option 1) Installing the new kernel directly on SD-card
 
Insert the SD card on your PC or Virtual Machine.
Check were the SD card is mounted with the lsblk command. 

```sh
~$ lsblk
sdb      8:16   1   3.7G  0 disk 
|-sdb1   8:17   1    50M  0 part /media/abc/BOOT
`-sdb2   8:18   1  1006M  0 part /media/abc/rootfs
```

In this case, the partition were mounted in /media/abc/BOOT and /media/abc/rootfs
 
 ```sh
~$ export BOOT=/media/abc/BOOT
~$ export ROOTFS=/media/abc/rootfs
```

Copy the files, the old kernel will be overwritten!

```sh
~$ cd linux-xlnx
~$ cp -v arch/arm/boot/uImage $BOOT
~$ cp -v devicetree.dtb $BOOT
```

In order to write into the rootfs we need to have root rights, and the ROOTFS variable was exported for the user.
We write an script or just hardcode the path to rootfs as following:

```sh
~$ sudo make modules_install INSTALL_MOD_PATH=$ROOTFS
```

Unmount the SD-card pratition to make sure files are written properly:

```sh
~$ sudo umount $BOOT
~$ sudo umount $ROOTFS
```

Insert the SD card in the board and let it boot.
Use the serial console to look for potential errors.
Once you are logged in, verify the used kernel version:

```sh
mircoZed:~# uname -a
mircoZed:~# dmesg
```

To check loaded modules:

```sh
microZed:~# lsmod
```

##(Option 2) Install the kernel over ssh
 
Once we have a kernel running it is usually faster to update it over ssh.

Mount "boot" partition

Skip this part if you have already mounted the boot partition in /boot.
To automatically mount the boot partition at boot time run the next command in the board:

```sh
microZed$ echo "/dev/mmcblk0p1 /boot auto noatime 1 2" >> /etc/fstab
```

Copy the kernel into the MicroZed.
Assuming that "boot" was mounted and the IP of the board is 192.168.1.103.
Copy the uImage and the device tree.

```sh
~$ scp arch/arm/boot/uImage  root@192.168.1.103:/media/ZYNQ/
~$ scp devicetree.dtb  root@192.168.1.103:/media/ZYNQ/
~$ scp -r mymodules/lib/modules/* root@192.168.1.103:/lib/modules/.
```

Safely reboot the board:

```sh
~$ ssh root@192.168.1.103 'sync; reboot'
```

###Install only one kernel module 

Example of coping the driver "drivers/remoteproc/zynq_remoteproc.ko" 

```sh
~$ make ARCH=arm modules -j2
```

Copy zynq_remoteproc.ko in the path /lib/modules/, x.y.z is the version of the kernel.

```sh
~$ scp drivers/remoteproc/zynq_remoteproc.ko root@192.168.1.103:/lib/modules/x.y.z/kernel/drivers/remoteproc/zynq_remoteproc.ko 
```
