##1. Get the Linux kernel sources for the Xilinx Zynq SoC

- Brief git tutorial
	http://www.wiki.xilinx.com/Using+Git

- Check for the latest official releases
    http://www.wiki.xilinx.com/Zynq+Releases


- Get the Linux kernel sources
	
1) Fetch sources:

```sh
export LINUX_TAG=xilinx-v2016.3
cd
wget https://github.com/Xilinx/linux-xlnx/archive/${LINUX_TAG}.tar.gz
mkdir -p $LINUX_TAG 
tar -zxvf $LINUX_TAG.tar.gz --strip-components=1 -C $LINUX_TAG
cd $LINUX_TAG
```
    
2) (Optional) Initialize a git repository to be able to track changes

```sh    
git init
git add .
git commit -m "Linux vanilla"
git branch $LINUX_TAG
git checkout $LINUX_TAG
```