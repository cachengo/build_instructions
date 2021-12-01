# Build for RK3399 ZAKU board

Recommended build host is Ubuntu 18.04 64-bit(amd64).

## Build environment setup

### Get the source code

Clone the repos:
    
    git clone -b mainline-zaku https://github.com/cachengo/symbioteos.git
    cd symbioteos
    git clone -b mainline-zaku git@github.com:cachengo/kernel
    git clone -b mainline-zaku https://github.com/cachengo/u-boot.git

You will get 

    build checkpoint_builds debos-build kernel out rkbin rkdeveloptool u-boot

### Install toolchain and other build tools

<pre>
  wget https://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz
  sudo tar xvf gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz  -C /usr/local/
  sudo apt-get install gcc-aarch64-linux-gnu device-tree-compiler libncurses5 libncurses5-dev build-essential libssl-dev mtools
  sudo apt-get install bc python dosfstools
  sudo apt-get install ruby-dev
  sudo gem install fpm
</pre>

### Setup environment variables

Add the following lines to the  ~/.bashrc file

<pre>
  export CROSS_COMPILE=/usr/local/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
  export PATH=/usr/local/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin:$PATH
</pre>

## Build u-boot and Flash spi

### Build u-boot

<pre>
  cd ~/symbioteos
  sudo ./build/mk-uboot.sh zaku
</pre>

The generated images will be copied to out/u-boot directory.

<pre>
  ls out/u-boot
  idbloader.img  rk3399_loader_v1.12.112.bin spi trust.img  uboot.img
</pre>

### Steps to Flash u-boot to spi

1. Connect usb to device (without SSD module) and to back of desktop (USB must be directly on motherboard)

2. Plug in power and hold 'maskrom' button on board (opposite end from the justice) for about 5 seconds

3. Run lsusb and confirm that rockchip device is connected (ID will be 2207:330c)

4. Initialize flash environment

<pre>
  cd ~/symbioteos
  sudo ./rkdeveloptool db out/u-boot/spi/rk3399_loader_spinor_v1.15.114.bin
</pre>

5. Flash u-boot

<pre>
  sudo ./rkdeveloptool wl 0 out/u-boot/spi/uboot-trust-spi.img
</pre>

6. Confirm u-boot has been flashed by rebooting board without ssd and using serial

## Build kernel and ZakuOS Image

### Package kernel and build ZakuOS

<pre>
  cd ~/symbioteos
  sudo ./build_image.sh
</pre>

<pre>
  ls debos-build/
  rk3399-zaku-ubuntu-bionic-server-arm64.img
</pre>

### Flash Image to NVMe ssd

1. Format NVMe ssd
2. Flash rk3399-zaku-ubuntu-bionic-server-arm64.img to ssd
3. Confirm that zaku boots with ssd
