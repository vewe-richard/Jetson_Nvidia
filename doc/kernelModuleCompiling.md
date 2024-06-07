
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













