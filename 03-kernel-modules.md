##3. How to compile and run a hello world kernel module in the MicroZedBoard

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

###Comiple the module

Be sure that you adapt the Makefile with:
```sh
    TARGET = 192.168.1.103                    ->  The ip address of the board
	TARGET_DIR = /root                          ->  where the driver will be copied
	KERNELDIR ?= ~/linux-xlnx                 ->  path to the linux kernel sources
```

And then build the module
```sh
~$ make 

A hello.ko file should be produced:

~$ ls
Makefile  Module.symvers  hello.c  hello.ko  hello.mod.c  hello.mod.o  hello.o  modules.order

Install the module:
~$ make install
```

###Connect to the board through SSH

```sh
~$ ssh root@192.168.1.103 
```

Go to /root and check that the *.ko is there

```sh
root@arm:~# cd /root
root@arm:~# ls
hello.ko
```

Insert the module and check that the hello world is loaded:

```sh
root@arm:~# insmod hello.ko
root@arm:~# lsmod

Check the kernel logs, a "hello world" should be visible:

root@arm:~# dmesg

To unload the module:

root@arm:~# rmmod hello
root@arm:~# dmesg
```
