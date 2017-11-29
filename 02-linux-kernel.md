##2. How to compile and install the Xilinx kernel on the Zybo board

###Prerequisites

- Linux PC or Virtual Machine with armhf toolchain installed
- 00-sd-card-setup: (Optional) Linux booting from SD card
- 01-xilinx-kernel-sources: Linux kernel sources from Xilinx

###Setup the environment for cross compiling

```sh
export CC=arm-linux-gnueabihf- 
export CROSS_COMPILE=${CC}
export LOADADDR=0x10008000
export ARCH=arm
```

Or use the script: 

```sh
source kernel_toolchain_env.sh
```
	
Check that gcc is found and the architecture is ARM:
```sh
${CC}gcc --version
```

We are now ready to compile the kernel.

The kernel developers provide use with Makefiles.
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

###Device tree

Copy the device tree from (this repo)/zynq-zybo.dts to (kernel sources)/arch/arm/boot/dts/. 

```sh
~$ gedit arch/arm/boot/dts/zynq-zybo.dts
```
Change the line "model" to:
```sh
	model = "Zynq ZYBO Development Board for ESP17";
```

To see all available peripherals look at the file:

(kernel sources)/arch/arm/boot/dts/zynq-7000.dtsi

###Compiling the kernel

We build kernel image, modules separately. Modules can be installed via make.
Adapt the core count of your processor in this case 2.

```sh
~/linux-xlnx$ make zynq-zybo.dtb
~/linux-xlnx$ make uImage -j2
~/linux-xlnx$ make modules -j2
~/linux-xlnx$ make modules_install INSTALL_MOD_PATH=./esp_modules 
```

###Manually compiling the Device tree (optional)

Compile the source into a device tree binary (dtb):

 ```sh
~$ dtc -I dts -O dtb -o devicetree.dtb arch/arm/boot/dts/zynq-zybo.dts 
 ```      

##Deploy to the board
    
### Local with an SD card reader

```sh
~$ cp -v arch/arm/boot/dts/zynb-zybo.dtb /media/$USER/BOOT/devicetree.dtb
~$ cp -v arch/arm/boot/uImage /media/$USER/BOOT/
~$ sudo rsync -avc esp_modules/lib/. /media/$USER/rootfs/lib/.
~$ sync

```   

Insert the SD card in the board and let it boot.
Use the serial console to look for potential errors.
Once you are logged in, verify the used kernel version:

```sh
zybo:~# uname -a
zybo:~# dmesg
```

To check loaded modules:

```sh
zybo:~# lsmod
```

###Install remotely over SSH

Once we have a kernel running it is usually faster to update it over ssh.

```sh
~$ export ip=192.168.1.103
~$ scp arch/arm/boot/dts/zynq-zybo.dtb root@$ip:/boot/devicetree.dtb
~$ scp arch/arm/boot/uImage root@$ip:/boot/
~$ rsync -avc esp_modules/lib/. root@$ip:/lib/. && ssh root@$ip 'sync'
```   
