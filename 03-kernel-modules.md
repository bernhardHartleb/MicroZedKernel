How to compile and run a hello world kernel module in the MicroZedBoard

##Prerequisites

- Linux PC or Virtual Machine with armhf toolchain installed
- 00-sd-card-setup: (Optional) Linux working on the microZed
- 01-xilinx-kernel-sources: Linux kernel sources from Xilinx


##Setup the environment for cross compiling

export CC=arm-linux-gnueabihf- 
export CROSS_COMPILE=${CC}
export LOADADDR=0x10008000
export ARCH=arm
	
Check that gcc is found and the architecture is ARM:
~$ ${CC}gcc --version

Settings for compiler:
---------------------
        Assuming an already installed arm-linux-gnueabihf
    
		~$ export CC=arm-linux-gnueabihf- 
		
        Check that is the linaro:

        ~$ ${CC}gcc --version
        arm-linux-gnueabihf-gcc (crosstool-NG linaro-1.13.1-4.9-2014.09 - Linaro GCC 4.9-2014.09) 4.9.2 20140904 (prerelease)
        Copyright (C) 2014 Free Software Foundation, Inc.
        This is free software; see the source for copying conditions.  There is NO
        warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.


Connect the board
-----------------

Connect the MicroZed over SSH, in this case the ip address is 192.168.1.103
    ssh root@192.168.1.103 

~$ cd src/hello_world

Be sure that you addapt the Makefile with:
    TARGET = 192.168.1.103                      ->  The ip address of the board
	TARGET_DIR = /root                          ->  where the driver will be copied
	KERNELDIR ?= ~/src/zynq/kernel/linux-xlnx   ->  path to the linux kernel sources

~$ make 

A hello.ko file should be produced:

~$ ls
Makefile  Module.symvers  hello.c  hello.ko  hello.mod.c  hello.mod.o  hello.o  modules.order

Install the module
~$ make install

Connect to the board through SSH

~$ ssh root@ip

Go to /root and check that the *.ko is there

root@arm:~# cd /root
root@arm:~# ls
hello.ko

Insert the module and check that the hello world is loaded:

root@arm:~# insmod hello.ko
root@arm:~# lsmod

Check the kernel logs, a "hello world" should be visible:

root@arm:~# dmesg

To unload the module:

root@arm:~# rmmod hello
root@arm:~# dmesg



