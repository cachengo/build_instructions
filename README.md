# Rockchip Debian SDK
    
Below are the instructions for how to build image for ROCK960.

## Requirement

Ubuntu x86_64

## Get the source code

Clone the repos:

    git clone -b release-4.4-zaku git@github.com:cachengo/kernel
    git clone git@github.com:cachengo/rkbin
    git clone -b rk3399-pie-zaku git@github.com:cachengo/u-boot
    git clone git@github.com:cachengo/build
    git clone git@github.com:rockchip-linux/rk-rootfs-build.git rootfs

You will get 

    build  kernel rkbin rootfs  u-boot

## Install toolchain and other build tools

    sudo apt-get install gcc-aarch64-linux-gnu device-tree-compiler libncurses5 libncurses5-dev build-essential libssl-dev mtools

## Build u-boot

    ./build/mk-uboot.sh zaku

The generated images will be copied to out/u-boot folder

    ls out/u-boot/
    idbloader.img  rk3399_loader_v1.08.106.bin  trust.img  uboot.img

## Build kernel

    ./build/mk-kernel.sh zaku

You will get the kernel image and dtb file

    ls out/kernel/
    Image  rock960-model-ab-linux.dtb rock960-model-c-linux.dtb

## Make rootfs image

Building a base debian system by ubuntu-build-service from linaro.

    cd rootfs
    sudo apt-get install binfmt-support qemu-user-static
    sudo dpkg -i ubuntu-build-service/packages/*        # ignore the broken dependencies, we will fix it next step
    sudo apt-get install -f
    RELEASE=stretch TARGET=desktop ARCH=armhf ./mk-base-debian.sh

This will bootstrap a Debian stretch image, you will get a rootfs tarball named `linaro-stretch-alip-xxxx.tar.gz`. 

Building the rk-debain rootfs with debug:

    VERSION=debug ARCH=armhf ./mk-rootfs-stretch.sh  && ./mk-image.sh

This will install Rockchip specified packages and hooks on the standard Debian rootfs and generate an ext4 format rootfs image at `rootfs/linaro-rootfs.img` .

## Combine everything into one image
    cd ..
    build/mk-image.sh -c rk3399 -t system -r rootfs/linaro-rootfs.img

This will combine u-boot, kernel and rootfs into one image and generate GPT partition table. Output is 

    out/system.img

