# Compile OP-TEE, U-Boot and Linux for RK3229


### Prequisites

- A host computer running a UNIX-like operating system
- An armhf toolchain
- GNU make
- git


### Prepare the build environment

Substitute *arm-linux-gnueabihf-* with your toolchain's target triplet, if needed.

```
$ mkdir build
$ build() { log=$1; shift 1; (date; echo; time make $@) 2>&1 | tee $log; }
$ export BUILD=`readlink -f build`
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabihf-
```


### Build OP-TEE

```
$ cd $BUILD
$ git clone https://github.com/OP-TEE/optee_os.git
$ cd optee_os
$ git checkout 3.5.0
$ build build-optee.log CFG_TEE_BENCHMARK=n CFG_TEE_CORE_LOG_LEVEL=3 DEBUG=1 PLATFORM=rockchip-rk322x -j2
```


### Build U-Boot

```
$ cd $BUILD
$ git clone git://git.denx.de/u-boot.git
$ cd u-boot
$ git checkout v2019.07-rc4
$ git checkout -b v2019.07-rc4/rk3229
$ patch -Np1 < ../../patch/u-boot/enable-arch-timer.patch 
```

*Apply the following patch only if your device does not have an eMMC, like the MXQ 4K and MXQ Pro 4K, or it has an eMMC that is not detected by U-Boot in its present state.*

```
$ patch -Np1 < ../../patch/u-boot/sdmmc-dm-pre-reloc.patch
```

```
$ cp ../optee_os/out/arm-plat-rockchip/core/tee-pager.bin tee.bin
$ cp ../../config/u-boot.config .config
$ make oldconfig
$ build build-uboot.log -j2 
$ build build-itb.log u-boot.itb -j2
$ tools/mkimage -n rk322x -T rksd -d tpl/u-boot-tpl.bin loader.img
$ cat spl/u-boot-spl.bin >> loader.img 
```


### Build Linux

```
$ cd $BUILD
$ git clone --branch v5.4 --depth 1 git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git 
$ cd linux
$ patch -Np1 < ../../patch/linux/0001-clk-rockchip-fix-incorrect-configuration-of-rk3228-a.patch
$ cp ../../config/linux.config .config
$ make oldconfig
$ build build-zImage.log -j2 zImage
$ build build-dtbs.log -j2 dtbs
$ build build-modules.log -j2 modules
```
