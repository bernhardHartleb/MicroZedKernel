##SD card setup for the our development board

Prompt:

    ~$ means your pc
    zybo$ the target Zybo board

###Setup the SD-card

A basic Debian image with gnu toolchain, go, perf and various debugging tools installed.
Download it and flash it in the SD card with the "dd" command. 

Download from Moodle:

compressed file: esp-debian-image-2016-11.img.tar.gz

uncompress
```sh
~$ tar -zxvf esp-debian-image-2016-11.img.tar.gz
```

With the command lsblk you can check to which device file is the SD card (/dev/sdX)

```sh
~$ lsblk
~$ sudo dd if=esp-debian-image-2016-11.img of=/dev/sdX bs=128k    -> be careful !
```

###Boot the Debian image

Set the jumpers to boot from the SD card.
Insert SD Card into the Zybo. Connect Zybo to the host with USB Serial port cable and RJ45 cable.
Reboot Zybo board by removing the power.

Check to witch driver is your serial port connected in the host:
~$ ls /dev/tty*
        should get something like ttyUSB0 

Use minicom or gtkterm at 115200 baud with the previous device.
If you press enter you should get a login prompt.

###Setup ethernet connection

The debian image has a static IP address (192.168.1.103) as a first step you can just give your desktop a static a IP as well.
You can run dhclient over the serial console if you board is connected to network with DHCP server.

```sh
~$ ifconfig eth0 192.168.1.101 up

# test with ping
~$ ping 192.168.1.103

# connect with ssh
~$ ssh root@192.168.1.103
```

User: debian
Password: debian

###Toolchain for cross compling

We will be using the armhf (hardware floating point support) toolchain which is already installed in your debian VM.

1)	The first step is to have a linux running in your board
2)	The second is to cross compile a user space program and run it in your board

```sh
zybo$ export CC=arm-linux-gnueabihf-
# Test
zybo$ ${CC}gcc --version
# Compile a hello world example
zybo$ ${CC}gcc hello-world.c -o hello-world.elf
```

###Copy files to your board

```sh
~$ scp hello-world.elf root@192.168.1.103:/root
~$ ssh root@192.168.1.103

# now we are in the Zed
zybo$ cd /root
zybo$ ./hello-world.elf
```
