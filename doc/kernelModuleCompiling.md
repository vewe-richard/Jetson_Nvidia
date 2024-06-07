
# Build kernel module for jetson 
  
Ref:
https://docs.nvidia.com/jetson/archives/r36.3/DeveloperGuide/SD/Kernel/KernelCustomization.html

## Set up kernel compiling environment in a container
```
cd ~
mkdir build
docker run -itd -v `pwd`/build:/root/build --name ubuntu ubuntu:20.04
docker exec -it ubuntu bash

apt update
apt install git-core build-essential bc wget vim flex bison libssl-dev kmod

# Install tool-chain
mkdir $HOME/l4t-gcc
cd $HOME/l4t-gcc
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/toolchain/aarch64--glibc--stable-2022.08-1.tar.bz2
tar xf aarch64--glibc--stable-2022.08-1.tar.bz2
export CROSS_COMPILE=$HOME/l4t-gcc/aarch64--glibc--stable-2022.08-1/bin/aarch64-buildroot-linux-gnu-

# Download source code
cd $HOME/build
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/release/jetson_linux_r36.3.0_aarch64.tbz2
tar xf jetson_linux_r36.3.0_aarch64.tbz2
cd $HOME/build/Linux_for_Tegra/source
./source_sync.sh -k -t jetson_36.3
```

## Building the Jetson Linux Kernel
```
docker exec -it ubuntu bash
export CROSS_COMPILE=$HOME/l4t-gcc/aarch64--glibc--stable-2022.08-1/bin/aarch64-buildroot-linux-gnu-

cd $HOME/build/Linux_for_Tegra/source
./generic_rt_build.sh "disable"
make -C kernel
```

## Build module ov428

Enter the container and create a directory to build ov428.c 
```
docker exec -it ubuntu bash

mkdir /root/build/ov428
cd /root/build/ov428
# copy ov428.c into the directory
```

Create Makefile with below contents
```text
obj-m := ov428.o

KDIR := /root/build/Linux_for_Tegra/source/kernel/kernel-jammy-src/
PWD := $(shell pwd)
ARCH := arm64

export CROSS_COMPILE=/root/l4t-gcc/aarch64--glibc--stable-2022.08-1/bin/aarch64-buildroot-linux-gnu-

all:
        $(MAKE) -C $(KDIR) M=$(PWD) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) modules

clean:
        $(MAKE) -C $(KDIR) M=$(PWD) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) clean
```

Make the module
```text
make
```

## Add module to image
```
docker exec -it ubuntu bash
cd /root/build/Linux_for_Tegra/source/kernel/kernel-jammy-src/drivers/staging
mkdir mymodule
cd mymodule
```
Create mymod.c with below contents
```
#include <linux/init.h>
#include <linux/module.h>

static int __init hello_init(void) {
    printk(KERN_INFO "Hello, Jetson! \n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, Jetson! \n");
}

module_init(hello_init);
module_exit(hello_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("test name");
MODULE_DESCRIPTION("A simple test module");

```
Create Makefile with below contents
```
obj-$(CONFIG_MYMODULE) += mymod.o
```
Create Kconfig with below contents
```
config MYMODULE
    tristate "my new modules"
    default m
```
Add the below contents to the  `/root/build/Linux_for_Tegra/source/kernel/kernel-jammy-src/drivers/staging/Makefile` file
```
obj-$(CONFIG_MYMODULE)     += mymodule/
```
Add the below contents to the  `/root/build/Linux_for_Tegra/source/kernel/kernel-jammy-src/drivers/staging/Kconfig` file
```
source "drivers/staging/mymodule/Kconfig"
```
Check and generate a new.config file
```
apt-get install libncurses5-dev
cd /root/build/Linux_for_Tegra/source/kernel/kernel-jammy-src/
make menuconfig
# goto Device Drivers -->
       Staging drivers -->
       you can see new module
       save and exit
```
Build kernel
```
export CROSS_COMPILE=$HOME/l4t-gcc/aarch64--glibc--stable-2022.08-1/bin/aarch64-buildroot-linux-gnu-
cd /root/build/Linux_for_Tegra/source
make -C kernel
```
Check new module
```
# ls /root/build/Linux_for_Tegra/source/kernel/kernel-jammy-src/drivers/staging/mymodule/
Kconfig  Makefile  modules.order  mymod.c  mymod.ko  mymod.mod  mymod.mod.c  mymod.mod.o  mymod.o
```

