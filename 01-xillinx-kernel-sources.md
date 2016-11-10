##Get the Linux kernel sources for the MicroZedboard

- Brief git tutorial
	http://www.wiki.xilinx.com/Using+Git

- Check for the latest official releases
    http://www.wiki.xilinx.com/Zynq+Releases
	
- Use the latest sources from Xilinx:
	
    1) Fetch sources: 

	```sh
    ~$ git clone git://github.com/Xilinx/linux-xlnx.git
    ~$ cd linux-xlnx
	
            
    2) Check out and switch to the desired branch:

	```sh
    ~$ git checkout -b esp-v2016.3 xilinx-v2016.3
	```