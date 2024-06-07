
# Build kernel module for jetson 

## Set up kernel compiling environment in a container
```
docker run -itd --name ubuntu ubuntu:20.04
docker exec -it ubuntu bash

# Next steps follow Nvidia developer guide below
# https://docs.nvidia.com/jetson/archives/r36.3/DeveloperGuide/SD/Kernel/KernelCustomization.html

apt install git-core build-essential bc wget vim flex bison libssl-dev kmod

# Install tool-chain
mkdir $HOME/l4t-gcc
cd $HOME/l4t-gcc
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/toolchain/aarch64--glibc--stable-2022.08-1.tar.bz2
tar xf aarch64--glibc--stable-2022.08-1.tar.bz2
export CROSS_COMPILE=$HOME/l4t-gcc/aarch64--glibc--stable-2022.08-1/bin/aarch64-buildroot-linux-gnu-

# Download source code
mkdir $HOME/build
cd $HOME/build
wget https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/sources/public_sources.tbz2
tar xf public_sources.tbz2
cd $HOME/build/Linux_for_Tegra/source
tar xf kernel_src.tbz2
tar xf kernel_oot_modules_src.tbz2
tar xf nvidia_kernel_display_driver_source.tbz2

# Building the Jetson Linux Kernel
cd $HOME/build/Linux_for_Tegra/source
# There were errors on building Out-of-Tree Modules if enable real-time kernel 
make -C kernel
'''

## Build module ov428
```
```
