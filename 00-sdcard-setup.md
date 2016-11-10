SD card setup for the MicroZedBoard

Prompt:

    ~$ means your pc
    microZed$ the MicroZed

Setup sd card:
--------------

A basic Debian image with gnu toolchain, go, perf and various debugging tools installed.
Download it and flash it in the SD card with the "dd" command. 

Download from Moodle:
# compressed file
esp-debian-image-2016-11.img.tar.gz

https://www.dropbox.com/s/fhpkdzlel790qbc/debian-small.img.tar.gz?dl=0
83366926e00c5f9e78ee620d52a43359  debian-small.img.tar.gz

# uncompress
tar -zxvf esp-debian-image-2016-11.img.tar.gz

With the command lsblk you can check to which device file is the SD card (/dev/sdX)

~$ lsblk
~$ sudo dd if=esp-debian-image-2016-11.img of=/dev/sdX bs=128k    -> be carefull!!

Boot the Debian image
--------------------------
Set the jumpers to boot from the SD card:
    JMP1 = "1"
    JMP2 = "0"
    JMP3 = "0"

Insert SD Card into MicroZed. Connect MicroZed to the host with USB
Serial port cable and RJ45 cable. Reboot MicroZed board by removing the power.

Check to witch driver is your serial port connected in the host:
~$ ls /dev/tty*
        should get something like ttyUSB0 

Use minicom or gtkterm at 115200 baud with the previous device.
If you press enter you should get a login prompt.

Setup ethernet connection
------------------------------
The debian image has a static IP address (192.168.1.103) as a first step you can just give your desktop a static a IP as well.
You can run dhclient over the serial console if you board is conntected to network with DHCP server.

~$ ifconfig eth0 192.168.1.101 up

# test with ping
~$ ping 192.168.1.103

# connect with ssh
~$ ssh root@192.168.1.103

User: debian
Password: debian

Toolchain for cross compling
--------------------------------
We will be using the armhf (hardware floating point support) toolchain which is already installed in your debian VM.

1)	The first step is to have a linux running in your MicroZed
2)	The second is to crosscompile an user space program and run it in your board

    microZed$ export CC=arm-linux-gnueabihf-

- Test

    microZed$ ${CC}gcc --version

- Compile a hello world example

    mircoZed$ ${CC}gcc hello-world.c -o hello-world.elf


Copying files to your board
-------------------------------

copy the elf file in your board:
~$ scp hello-world.elf root@192.168.1.103:/root
~$ ssh root@192.168.1.103

# now we are in the Zed
microZedt$ cd /root
microZed$ ./hello-world.elf
