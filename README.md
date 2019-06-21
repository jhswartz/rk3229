# RK3229 Unyoked

### Prequisites

- An RK3229 based device
- An SD card reader/writer 
- A host computer with a UNIX-like operating system
- Access to the appropriate UART RX, TX and GND lines
- A serial communications program for UART access
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
$ patch -Np1 < ../../patch/u-boot/sdmmc-dm-pre-reloc.patch 
$ cp ../optee_os/out/arm-plat-rockchip/core/tee-pager.bin tee.bin
$ cp ../../config/u-boot.config .config
$ make oldconfig
$ build build-uboot.log -j2 
$ build build-itb.log u-boot.itb -j2
$ tools/mkimage -n rk322x -T rksd -d tpl/u-boot-tpl.bin loader.img
$ cat spl/u-boot-spl.bin >> loader.img 
```

### Install U-Boot

Replace *mmcblk0* with the actual block device associated with your SD card.

```
# cd $BUILD/u-boot
# dd if=/dev/zero of=/dev/mmcblk0 bs=4M count=3
# dd if=loader.img of=/dev/mmcblk0 bs=512 seek=64
# dd if=u-boot.itb of=/dev/mmcblk0 bs=512 seek=16384
```

### Test boot from SD Card

Insert the SD card, connect to UART2 (1500000 baud 8N1), and supply power to the device.
You may need to clear the eMMC's loader at sector 64 to boot from an SD card.

```
# dterm /dev/ttyUSB0 1500000 8 n 1
TPL InitReturning to boot ROM...

U-Boot SPL 2019.07-rc4-dirty (Jun 21 2019 - 09:01:56 +0000)
Trying to boot from MMC1
D/TC:0 0 add_phys_mem:576 VCORE_UNPG_RX_PA type TEE_RAM_RX 0x68400000 size 0x00052000
D/TC:0 0 add_phys_mem:576 VCORE_UNPG_RW_PA type TEE_RAM_RW 0x68452000 size 0x000ae000
D/TC:0 0 add_phys_mem:576 TA_RAM_START type TA_RAM 0x68500000 size 0x00100000
D/TC:0 0 add_phys_mem:576 TEE_SHMEM_START type NSEC_SHM 0x68600000 size 0x00100000
D/TC:0 0 add_phys_mem:576 ROUNDDOWN(0x10100000, CORE_MMU_PGDIR_SIZE) type IO_SEC 0x10100000 size 0x22000000
D/TC:0 0 add_phys_mem:576 ROUNDDOWN(0x10080000, CORE_MMU_PGDIR_SIZE) type IO_NSEC 0x10000000 size 0x00100000
D/TC:0 0 verify_special_mem_areas:514 No NSEC DDR memory area defined
D/TC:0 0 add_va_space:615 type RES_VASPACE size 0x00a00000
D/TC:0 0 add_va_space:615 type SHM_VASPACE size 0x02000000
D/TC:0 0 dump_mmap_table:751 type TEE_RAM_RX   va 0x68400000..0x68451fff pa 0x68400000..0x68451fff size 0x00052000 (smallpg)
D/TC:0 0 dump_mmap_table:751 type TEE_RAM_RW   va 0x68452000..0x684fffff pa 0x68452000..0x684fffff size 0x000ae000 (smallpg)
D/TC:0 0 dump_mmap_table:751 type SHM_VASPACE  va 0x68500000..0x6a4fffff pa 0x00000000..0x01ffffff size 0x02000000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type IO_SEC       va 0x6a500000..0x8c4fffff pa 0x10100000..0x320fffff size 0x22000000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type RES_VASPACE  va 0x8c500000..0x8cefffff pa 0x00000000..0x009fffff size 0x00a00000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type TA_RAM       va 0x8cf00000..0x8cffffff pa 0x68500000..0x685fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type NSEC_SHM     va 0x8d000000..0x8d0fffff pa 0x68600000..0x686fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type IO_NSEC      va 0x8d100000..0x8d1fffff pa 0x10000000..0x100fffff size 0x00100000 (pgdir)
D/TC:0 0 core_mmu_alloc_l2:273 L2 table used: 1/4
I/TC: 
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[0] with top at 0x6847a838
D/TC:0 0 init_canaries:173 watch *0x6847a83c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[1] with top at 0x6847af78
D/TC:0 0 init_canaries:173 watch *0x6847af7c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[2] with top at 0x6847b6b8
D/TC:0 0 init_canaries:173 watch *0x6847b6bc
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[3] with top at 0x6847bdf8
D/TC:0 0 init_canaries:173 watch *0x6847bdfc
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[0] with top at 0x6847c638
D/TC:0 0 init_canaries:174 watch *0x6847c63c
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[1] with top at 0x6847ce78
D/TC:0 0 init_canaries:174 watch *0x6847ce7c
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[2] with top at 0x6847d6b8
D/TC:0 0 init_canaries:174 watch *0x6847d6bc
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[3] with top at 0x6847def8
D/TC:0 0 init_canaries:174 watch *0x6847defc
D/TC:0 0 init_canaries:176 #Stack canaries for stack_thread[0] with top at 0x6847ff38
D/TC:0 0 init_canaries:176 watch *0x6847ff3c
D/TC:0 0 init_canaries:176 #Stack canaries for stack_thread[1] with top at 0x68481f78
D/TC:0 0 init_canaries:176 watch *0x68481f7c
I/TC: OP-TEE version: 3.5.0-143-ga30ddda9 #1 Thu Jun 20 12:18:57 UTC 2019 arm
D/TC:0 0 check_ta_store:653 TA store: "Secure Storage TA"
D/TC:0 0 check_ta_store:653 TA store: "REE [buffered]"
D/TC:0 0 mobj_mapped_shm_init:447 Shared memory address range: 68500000, 6a500000
I/TC: Initialized
D/TC:0 0 init_primary_helper:1105 Primary CPU switching to normal world boot


U-Boot 2019.07-rc4-dirty (Jun 21 2019 - 09:05:43 +0000)

Model: Rockchip RK3229 Evaluation board
DRAM:  1022 MiB
MMC:   dwmmc@30000000: 1, dwmmc@30020000: 0
Loading Environment from MMC... *** Warning - bad CRC, using default environment

In:    serial@11030000
Out:   serial@11030000
Err:   serial@11030000
Model: Rockchip RK3229 Evaluation board
rockchip_dnl_key_pressed: adc_channel_single_shot fail!
Net:   
Warning: ethernet@30200000 (eth0) using random MAC address - 3e:9d:1e:46:79:fa
eth0: ethernet@30200000
Hit any key to stop autoboot:  2
```

Be sure to stop autoboot!

### Partition the SD Card

```
=> mmc list
dwmmc@30000000: 1
dwmmc@30020000: 0 (eMMC)
=> mmc info
Device: dwmmc@30000000
Manufacturer ID: 28
OEM: 4245
Name: 00000 
Bus Speed: 50000000
Mode: SD High Speed (50MHz)
Rd Block Len: 512
SD version 3.0
High Capacity: Yes
Capacity: 14.9 GiB
Bus Width: 4-bit
Erase Group Size: 512 Bytes
=> mmc dev 1
switch to partitions #0, OK
mmc1 is current device
=> gpt write mmc 1 $partitions
Writing GPT: success!
=> gpt verify mmc 1 $partitions
Verify GPT: success!
=> mmc part

Partition Map for MMC device 1  --   Partition Type: EFI

Part    Start LBA       End LBA         Name
        Attributes
        Type GUID
        Partition GUID
  1     0x00000040      0x00001f7f      "loader1"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   34c35b76-95c9-6a41-86af-8f765f638e16
  2     0x00004000      0x00005fff      "loader2"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   285b2b7b-8419-ef42-a4ea-3d634b8fd4c0
  3     0x00006000      0x00007fff      "trust"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   9e403b9f-3712-4a4a-8eab-f455b40084e5
  4     0x00008000      0x0003ffff      "boot"
        attrs:  0x0000000000000004
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   105993e9-92d4-4841-81d0-18b51a15a316
  5     0x00040000      0x01db3fde      "rootfs"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   69dad710-2ce4-4e3c-b16c-21a1d49abed3
```

Disconnect power from the device and remove the SD card.

### Prepare the boot filesystem

```
# cd $BUILD
# mkfs.vfat /dev/mmcblk0p4
# BOOT=/mnt/mmcblk0p4
# mkdir $BOOT
# mount /dev/mmcblk0p4 $BOOT
# mkdir $BOOT/dtb
# cp linux/arch/arm/boot/dts/rk3229-xms6.dtb $BOOT/dtb
# cp linux/arch/arm/boot/zImage $BOOT
# mkdir $BOOT/extlinux
# cat > $BOOT/extlinux/extlinux.conf
timeout 5
label default
        fdt /dtb/rk3229-xms6.dtb
        linux /zImage
        append console=ttyS2,1500000n8 console=tty0 video=HDMI-A-1:1920x1080@60 ignore_loglevel debug rw root=/dev/mmcblk0p5 rootfstype=ext2 rootwait
[^D]
# umount $BOOT
```

### Prepare the rootfs

```
# mkfs.ext2 /dev/mmcblk0p5
# ROOTFS=/mnt/mmcblk0p5
# mkdir $ROOTFS
# mount /dev/mmcblk0p5 $ROOTFS
```

Copy your desired distribution to $ROOTFS, or use the following commands to bootstrap the current stable release of Devuan instead.

```
# qemu-debootstrap --arch armhf stable $ROOTFS http://pkgmaster.devuan.org/merged
# chroot $ROOTFS /bin/bash
# passwd root
# > /etc/motd
# > /etc/issue
# > /etc/issue.net
# echo "workstation" > /etc/hostname
# echo "T2:23:respawn:/sbin/getty -L ttyS2 1500000 linux" >> /etc/inittab
# cat > /etc/fstab
/dev/mmcblk0p4	/boot			vfat		defaults	0	0
none		/tmp			tmpfs		defaults	0	0
none		/sys/kernel/debug	debugfs		defaults	0	0
[^D]
# exit
```

### Build Linux

```
$ cd $BUILD
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git 
$ cd linux
$ git checkout v5.2-rc5
$ git checkout -b v5.2-rc5/rk3229
$ patch -Np1 < ../../patch/linux/clk-rockchip-add-1.464GHz-cpu-clock-rate-to-rk3228.patch
$ patch -Np1 < ../../patch/linux/drm-rockchip-dw_hdmi-add-basic-rk3228-support.patch
$ patch -Np1 < ../../patch/linux/clk-rockchip-add-clock-id-for-hdmi_phy-special-clock.patch
$ patch -Np1 < ../../patch/linux/clk-rockchip-export-HDMIPHY-clock.patch
$ patch -Np1 < ../../patch/linux/ARM-dts-rockchip-add-display-nodes-for-rk322x.patch
$ patch -Np1 < ../../patch/linux/ARM-dts-rockchip-fix-vop-iommu-cells-on-rk322x.patch
$ patch -Np1 < ../../patch/linux/ARM-dts-add-device-tree-for-Mecer-Xtreme-Mini-S6.patch
$ cp ../../config/linux.config .config
$ make oldconfig
$ build build-zImage.log -j2 zImage                                                                                                                                               
$ build build-dtbs.log -j2 dtbs
$ build build-modules.log -j2 modules
# build build-modules_install.log -j2 INSTALL_MOD_PATH=$ROOTFS modules_install
# umount $ROOTFS
```

### Boot

```
# dterm /dev/ttyUSB0 1500000 8 n 1
TPL InitReturning to boot ROM...

U-Boot SPL 2019.07-rc3-00112-g6d277fb0ed-dirty (Jun 20 2019 - 13:53:19 +0000)
Trying to boot from MMC1
D/TC:0 0 add_phys_mem:576 VCORE_UNPG_RX_PA type TEE_RAM_RX 0x68400000 size 0x00052000
D/TC:0 0 add_phys_mem:576 VCORE_UNPG_RW_PA type TEE_RAM_RW 0x68452000 size 0x000ae000
D/TC:0 0 add_phys_mem:576 TA_RAM_START type TA_RAM 0x68500000 size 0x00100000
D/TC:0 0 add_phys_mem:576 TEE_SHMEM_START type NSEC_SHM 0x68600000 size 0x00100000
D/TC:0 0 add_phys_mem:576 ROUNDDOWN(0x10100000, CORE_MMU_PGDIR_SIZE) type IO_SEC 0x10100000 size 0x22000000
D/TC:0 0 add_phys_mem:576 ROUNDDOWN(0x10080000, CORE_MMU_PGDIR_SIZE) type IO_NSEC 0x10000000 size 0x00100000
D/TC:0 0 verify_special_mem_areas:514 No NSEC DDR memory area defined
D/TC:0 0 add_va_space:615 type RES_VASPACE size 0x00a00000
D/TC:0 0 add_va_space:615 type SHM_VASPACE size 0x02000000
D/TC:0 0 dump_mmap_table:751 type TEE_RAM_RX   va 0x68400000..0x68451fff pa 0x68400000..0x68451fff size 0x00052000 (smallpg)
D/TC:0 0 dump_mmap_table:751 type TEE_RAM_RW   va 0x68452000..0x684fffff pa 0x68452000..0x684fffff size 0x000ae000 (smallpg)
D/TC:0 0 dump_mmap_table:751 type SHM_VASPACE  va 0x68500000..0x6a4fffff pa 0x00000000..0x01ffffff size 0x02000000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type IO_SEC       va 0x6a500000..0x8c4fffff pa 0x10100000..0x320fffff size 0x22000000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type RES_VASPACE  va 0x8c500000..0x8cefffff pa 0x00000000..0x009fffff size 0x00a00000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type TA_RAM       va 0x8cf00000..0x8cffffff pa 0x68500000..0x685fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type NSEC_SHM     va 0x8d000000..0x8d0fffff pa 0x68600000..0x686fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:751 type IO_NSEC      va 0x8d100000..0x8d1fffff pa 0x10000000..0x100fffff size 0x00100000 (pgdir)
D/TC:0 0 core_mmu_alloc_l2:273 L2 table used: 1/4
I/TC: 
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[0] with top at 0x6847a838
D/TC:0 0 init_canaries:173 watch *0x6847a83c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[1] with top at 0x6847af78
D/TC:0 0 init_canaries:173 watch *0x6847af7c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[2] with top at 0x6847b6b8
D/TC:0 0 init_canaries:173 watch *0x6847b6bc
D/TC:0 0 init_canaries:173 #Stack canaries for stack_tmp[3] with top at 0x6847bdf8
D/TC:0 0 init_canaries:173 watch *0x6847bdfc
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[0] with top at 0x6847c638
D/TC:0 0 init_canaries:174 watch *0x6847c63c
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[1] with top at 0x6847ce78
D/TC:0 0 init_canaries:174 watch *0x6847ce7c
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[2] with top at 0x6847d6b8
D/TC:0 0 init_canaries:174 watch *0x6847d6bc
D/TC:0 0 init_canaries:174 #Stack canaries for stack_abt[3] with top at 0x6847def8
D/TC:0 0 init_canaries:174 watch *0x6847defc
D/TC:0 0 init_canaries:176 #Stack canaries for stack_thread[0] with top at 0x6847ff38
D/TC:0 0 init_canaries:176 watch *0x6847ff3c
D/TC:0 0 init_canaries:176 #Stack canaries for stack_thread[1] with top at 0x68481f78
D/TC:0 0 init_canaries:176 watch *0x68481f7c
I/TC: OP-TEE version: 3.5.0-143-ga30ddda9 #1 Thu Jun 20 12:18:57 UTC 2019 arm
D/TC:0 0 check_ta_store:653 TA store: "Secure Storage TA"
D/TC:0 0 check_ta_store:653 TA store: "REE [buffered]"
D/TC:0 0 mobj_mapped_shm_init:447 Shared memory address range: 68500000, 6a500000
I/TC: Initialized
D/TC:0 0 init_primary_helper:1105 Primary CPU switching to normal world boot


U-Boot 2019.07-rc3-00112-g6d277fb0ed-dirty (Jun 20 2019 - 13:53:46 +0000)

Model: Rockchip RK3229 Evaluation board
DRAM:  1022 MiB
MMC:   dwmmc@30000000: 1, dwmmc@30020000: 0
Loading Environment from MMC... *** Warning - bad CRC, using default environment

In:    serial@11030000
Out:   serial@11030000
Err:   serial@11030000
Model: Rockchip RK3229 Evaluation board
rockchip_dnl_key_pressed: adc_channel_single_shot fail!
Net:   
Warning: ethernet@30200000 (eth0) using random MAC address - 3e:9d:1e:46:79:fa
eth0: ethernet@30200000
Hit any key to stop autoboot:  2  1  0 
switch to partitions #0, OK
mmc1 is current device
Scanning mmc 1:4...
Found /extlinux/extlinux.conf
Retrieving file: /extlinux/extlinux.conf
209 bytes read in 2 ms (101.6 KiB/s)
1:	default
Retrieving file: /zImage
5297016 bytes read in 242 ms (20.9 MiB/s)
append: console=ttyS2,1500000n8 console=tty0 video=HDMI-A-1:1920x1080@60 ignore_loglevel debug rw root=/dev/mmcblk0p5 rootfstype=ext2 rootwait
Retrieving file: /dtb/rk3229-xms6.dtb
23615 bytes read in 6 ms (3.8 MiB/s)
## Flattened Device Tree blob at 61f00000
   Booting using the fdt blob at 0x61f00000
   Loading Device Tree to 683f7000, end 683ffc3e ... OK

Starting kernel ...

D/TC:0   psci_cpu_on:278 core_id: 1
D/TC:1   init_secondary_helper:1129 Secondary CPU Switching to normal world boot
D/TC:0   psci_cpu_on:278 core_id: 2
D/TC:2   init_secondary_helper:1129 Secondary CPU Switching to normal world boot
D/TC:0   psci_cpu_on:278 core_id: 3
D/TC:3   init_secondary_helper:1129 Secondary CPU Switching to normal world boot
[    0.000000] Booting Linux on physical CPU 0xf00
[    0.000000] Linux version 5.2.0-rc5-dirty (justin@v01.28459.vpscontrol.net) (gcc version 6.3.0 20170516 (Debian 6.3.0-18)) #1 SMP Thu Jun 20 11:52:01 UTC 2019
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Rockchip RK3229 (Mecer Xtreme Mini S6)
[    0.000000] printk: debug: ignoring loglevel setting.
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] cma: Reserved 64 MiB at 0x8c000000
[    0.000000] On node 0 totalpages: 261632
[    0.000000]   Normal zone: 1536 pages used for memmap
[    0.000000]   Normal zone: 0 pages reserved
[    0.000000]   Normal zone: 196096 pages, LIFO batch:63
[    0.000000]   HighMem zone: 65536 pages, LIFO batch:15
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv1.0 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: MIGRATE_INFO_TYPE not supported.
[    0.000000] psci: SMC Calling Convention v1.0
[    0.000000] percpu: Embedded 20 pages/cpu s51788 r8192 d21940 u81920
[    0.000000] pcpu-alloc: s51788 r8192 d21940 u81920 alloc=20*4096
[    0.000000] pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 260096
[    0.000000] Kernel command line: console=ttyS2,1500000n8 console=tty0 video=HDMI-A-1:1920x1080@60 ignore_loglevel debug rw root=/dev/mmcblk0p5 rootfstype=ext2 rootwait
[    0.000000] Dentry cache hash table entries: 131072 (order: 7, 524288 bytes)
[    0.000000] Inode-cache hash table entries: 65536 (order: 6, 262144 bytes)
[    0.000000] Memory: 958336K/1046528K available (8192K kernel code, 549K rwdata, 2084K rodata, 1024K init, 267K bss, 22656K reserved, 65536K cma-reserved, 262144K highmem)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu: 	RCU event tracing is enabled.
[    0.000000] rcu: 	RCU restricting CPUs from NR_CPUS=16 to nr_cpu_ids=4.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000000] random: get_random_bytes called from start_kernel+0x304/0x494 with crng_init=0
[    0.000000] rockchip_mmc_get_phase: invalid clk rate
[    0.000000] rockchip_mmc_get_phase: invalid clk rate
[    0.000000] rockchip_mmc_get_phase: invalid clk rate
[    0.000000] rockchip_mmc_get_phase: invalid clk rate
[    0.000000] rockchip_mmc_get_phase: invalid clk rate
[    0.000000] rockchip_mmc_get_phase: invalid clk rate
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x588fe9dc0, max_idle_ns: 440795202592 ns
[    0.000008] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398046511097ns
[    0.000023] Switching to timer-based delay loop, resolution 41ns
[    0.000987] Console: colour dummy device 80x30
[    0.001501] printk: console [tty0] enabled
[    0.001563] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=240000)
[    0.001596] pid_max: default: 32768 minimum: 301
[    0.001788] Mount-cache hash table entries: 2048 (order: 1, 8192 bytes)
[    0.001817] Mountpoint-cache hash table entries: 2048 (order: 1, 8192 bytes)
[    0.002523] *** VALIDATE proc ***
[    0.002697] *** VALIDATE cgroup1 ***
[    0.002722] *** VALIDATE cgroup2 ***
[    0.002743] CPU: Testing write buffer coherency: ok
[    0.003267] CPU0: update cpu_capacity 1024
[    0.003301] CPU0: thread -1, cpu 0, socket 15, mpidr 80000f00
[    0.004020] Setting up static identity map for 0x60100000 - 0x60100060
[    0.004193] rcu: Hierarchical SRCU implementation.
[    0.007849] smp: Bringing up secondary CPUs ...
[    0.010443] CPU1: update cpu_capacity 1024
[    0.010453] CPU1: thread -1, cpu 1, socket 15, mpidr 80000f01
[    0.013090] CPU2: update cpu_capacity 1024
[    0.013098] CPU2: thread -1, cpu 2, socket 15, mpidr 80000f02
[    0.015620] CPU3: update cpu_capacity 1024
[    0.015628] CPU3: thread -1, cpu 3, socket 15, mpidr 80000f03
[    0.015768] smp: Brought up 1 node, 4 CPUs
[    0.015873] SMP: Total of 4 processors activated (192.00 BogoMIPS).
[    0.015889] CPU: All CPU(s) started in SVC mode.
[    0.017252] devtmpfs: initialized
[    0.023883] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 5
[    0.024256] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.024313] futex hash table entries: 1024 (order: 4, 65536 bytes)
[    0.027099] pinctrl core: initialized pinctrl subsystem
[    0.028661] NET: Registered protocol family 16
[    0.031516] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.033343] cpuidle: using governor menu
[    0.033837] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.033871] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.034527] Serial: AMBA PL011 UART driver
[    0.066730] vcc_sys: supplied by dc_12v
[    0.067154] vccio_1v8: supplied by vcc_sys
[    0.067406] vcc_host: supplied by vcc_sys
[    0.067664] vccio_3v3: supplied by vcc_sys
[    0.067902] vcc_phy: supplied by vccio_1v8
[    0.069483] SCSI subsystem initialized
[    0.069778] usbcore: registered new interface driver usbfs
[    0.069855] usbcore: registered new interface driver hub
[    0.069998] usbcore: registered new device driver usb
[    0.070445] pps_core: LinuxPPS API ver. 1 registered
[    0.070474] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.070535] EDAC MC: Ver: 3.0.0
[    0.073029] clocksource: Switched to clocksource arch_sys_counter
[    0.132917] NET: Registered protocol family 2
[    0.134087] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 6144 bytes)
[    0.134153] TCP established hash table entries: 8192 (order: 3, 32768 bytes)
[    0.134266] TCP bind hash table entries: 8192 (order: 4, 65536 bytes)
[    0.134418] TCP: Hash tables configured (established 8192 bind 8192)
[    0.134612] UDP hash table entries: 512 (order: 2, 16384 bytes)
[    0.134695] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes)
[    0.134989] NET: Registered protocol family 1
[    0.135054] PCI: CLS 0 bytes, default 64
[    0.136323] hw perfevents: enabled with armv7_cortex_a7 PMU driver, 5 counters available
[    0.138516] Initialise system trusted keyrings
[    0.138884] workingset: timestamp_bits=30 max_order=18 bucket_order=0
[    0.146740] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.146998] ntfs: driver 2.1.32 [Flags: R/O].
[    0.147784] Key type asymmetric registered
[    0.147816] Asymmetric key parser 'x509' registered
[    0.147901] bounce: pool size: 64 pages
[    0.147971] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)
[    0.147997] io scheduler mq-deadline registered
[    0.148012] io scheduler kyber registered
[    0.155670] pwm-regulator: supplied by vcc_sys
[    0.156274] pwm-regulator: supplied by vcc_sys
[    0.222175] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.225480] printk: console [ttyS2] disabled
[    0.225582] 11030000.serial: ttyS2 at MMIO 0x11030000 (irq = 28, base_baud = 1500000) is a 16550A
[    0.285940] printk: console [ttyS2] enabled
[    0.288194] rockchip-vop 20050000.vop: Adding to iommu group 0
[    0.292052] rockchip-drm display-subsystem: bound 20050000.vop (ops 0xc0950168)
[    0.292974] dwhdmi-rockchip 200a0000.hdmi: Detected HDMI TX controller v2.01a with HDCP (inno_dw_hdmi_phy2)
[    0.294480] dwhdmi-rockchip 200a0000.hdmi: registered DesignWare HDMI I2C bus driver
[    0.295590] rockchip-drm display-subsystem: bound 200a0000.hdmi (ops 0xc0953270)
[    0.296268] [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
[    0.296859] [drm] No driver support for vblank timestamp query.
[    0.406546] random: fast init done
[    0.485932] Console: switching to colour frame buffer device 240x67
[    0.536646] rockchip-drm display-subsystem: fb0: rockchipdrmfb frame buffer device
[    0.538556] [drm] Initialized rockchip 1.0.0 20140818 for display-subsystem on minor 0
[    0.541538] lima 20000000.gpu: bus rate = 297000000
[    0.542244] lima 20000000.gpu: mod rate = 297000000
[    0.543861] lima 20000000.gpu: gp - mali400 version major 1 minor 1
[    0.544758] lima 20000000.gpu: pp0 - mali400 version major 1 minor 1
[    0.545663] lima 20000000.gpu: pp1 - mali400 version major 1 minor 1
[    0.546452] lima 20000000.gpu: l2 cache 64K, 4-way, 64byte cache line, 64bit external bus
[    0.549047] [drm] Initialized lima 1.0.0 20190217 for 20000000.gpu on minor 1
[    0.567682] brd: module loaded
[    0.584112] loop: module loaded
[    0.586369] libphy: Fixed MDIO Bus: probed
[    0.587893] rk_gmac-dwmac 30200000.ethernet: PTP uses main clock
[    0.588856] rk_gmac-dwmac 30200000.ethernet: clock input or output? (output).
[    0.589734] rk_gmac-dwmac 30200000.ethernet: Can not read property: tx_delay.
[    0.590591] rk_gmac-dwmac 30200000.ethernet: set tx_delay to 0x30
[    0.591328] rk_gmac-dwmac 30200000.ethernet: Can not read property: rx_delay.
[    0.592184] rk_gmac-dwmac 30200000.ethernet: set rx_delay to 0x10
[    0.592931] rk_gmac-dwmac 30200000.ethernet: integrated PHY? (yes).
[    0.594009] rk_gmac-dwmac 30200000.ethernet: cannot get clock clk_mac_speed
[    0.600613] rk_gmac-dwmac 30200000.ethernet: init for RMII
[    0.643468] rk_gmac-dwmac 30200000.ethernet: User ID: 0x10, Synopsys ID: 0x35
[    0.644392] rk_gmac-dwmac 30200000.ethernet: 	DWMAC1000
[    0.645038] rk_gmac-dwmac 30200000.ethernet: DMA HW capability register supported
[    0.645937] rk_gmac-dwmac 30200000.ethernet: RX Checksum Offload Engine supported
[    0.646834] rk_gmac-dwmac 30200000.ethernet: COE Type 2
[    0.647469] rk_gmac-dwmac 30200000.ethernet: TX Checksum insertion supported
[    0.648308] rk_gmac-dwmac 30200000.ethernet: Wake-Up On Lan supported
[    0.649145] rk_gmac-dwmac 30200000.ethernet: Normal descriptors
[    0.649862] rk_gmac-dwmac 30200000.ethernet: Ring mode enabled
[    0.650565] rk_gmac-dwmac 30200000.ethernet: Enable RX Mitigation via HW Watchdog Timer
[    0.651544] rk_gmac-dwmac 30200000.ethernet (unnamed net_device) (uninitialized): device MAC address 5e:c6:77:59:d5:e1
[    0.653190] libphy: stmmac: probed
[    0.656308] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.657243] ehci-pci: EHCI PCI platform driver
[    0.657895] ehci-platform: EHCI generic platform driver
[    0.661093] ehci-platform 30080000.usb: EHCI Host Controller
[    0.672108] ehci-platform 30080000.usb: new USB bus registered, assigned bus number 1
[    0.683584] ehci-platform 30080000.usb: irq 38, io mem 0x30080000
[    0.723063] ehci-platform 30080000.usb: USB 2.0 started, EHCI 1.00
[    0.735568] hub 1-0:1.0: USB hub found
[    0.746297] hub 1-0:1.0: 1 port detected
[    0.760262] ehci-platform 300c0000.usb: EHCI Host Controller
[    0.771140] ehci-platform 300c0000.usb: new USB bus registered, assigned bus number 2
[    0.782424] ehci-platform 300c0000.usb: irq 40, io mem 0x300c0000
[    0.823045] ehci-platform 300c0000.usb: USB 2.0 started, EHCI 1.00
[    0.835151] hub 2-0:1.0: USB hub found
[    0.845773] hub 2-0:1.0: 1 port detected
[    0.859626] ehci-platform 30100000.usb: EHCI Host Controller
[    0.870509] ehci-platform 30100000.usb: new USB bus registered, assigned bus number 3
[    0.882361] ehci-platform 30100000.usb: irq 42, io mem 0x30100000
[    0.923067] ehci-platform 30100000.usb: USB 2.0 started, EHCI 1.00
[    0.935140] hub 3-0:1.0: USB hub found
[    0.945714] hub 3-0:1.0: 1 port detected
[    0.957042] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.967680] ohci-platform: OHCI generic platform driver
[    0.978616] ohci-platform 300a0000.usb: Generic Platform OHCI controller
[    0.989223] ohci-platform 300a0000.usb: new USB bus registered, assigned bus number 4
[    1.000643] ohci-platform 300a0000.usb: irq 39, io mem 0x300a0000
[    1.078475] hub 4-0:1.0: USB hub found
[    1.088665] hub 4-0:1.0: 1 port detected
[    1.099652] ohci-platform 300e0000.usb: Generic Platform OHCI controller
[    1.109929] ohci-platform 300e0000.usb: new USB bus registered, assigned bus number 5
[    1.120622] ohci-platform 300e0000.usb: irq 41, io mem 0x300e0000
[    1.198454] hub 5-0:1.0: USB hub found
[    1.208530] hub 5-0:1.0: 1 port detected
[    1.219359] ohci-platform 30120000.usb: Generic Platform OHCI controller
[    1.229314] ohci-platform 30120000.usb: new USB bus registered, assigned bus number 6
[    1.239777] ohci-platform 30120000.usb: irq 43, io mem 0x30120000
[    1.318613] hub 6-0:1.0: USB hub found
[    1.328524] hub 6-0:1.0: 1 port detected
[    1.340015] usbcore: registered new interface driver usb-storage
[    1.350542] i2c /dev entries driver
[    1.363596] rockchip-thermal 11150000.tsadc: Missing tshut-polarity property, using default (low)
[    1.374150] rockchip-thermal 11150000.tsadc: Missing rockchip,grf property
[    1.385830] softdog: initialized. soft_noboot=0 soft_margin=60 sec soft_panic=0 (nowayout=0)
[    1.396691] Synopsys Designware Multimedia Card Interface Driver
[    1.407962] dwmmc_rockchip 30000000.dwmmc: IDMAC supports 32-bit address mode.
[    1.418707] dwmmc_rockchip 30000000.dwmmc: Using internal DMA controller.
[    1.429138] dwmmc_rockchip 30000000.dwmmc: Version ID is 270a
[    1.439343] dwmmc_rockchip 30000000.dwmmc: DW MMC controller at irq 36,32 bit host data width,256 deep fifo
[    1.463313] mmc_host mmc0: Bus speed (slot 0) = 400000Hz (slot req 400000Hz, actual 400000HZ div = 0)
[    1.488688] ledtrig-cpu: registered to indicate activity on CPUs
[    1.499936] usbcore: registered new interface driver usbhid
[    1.510630] usbhid: USB HID core driver
[    1.521769] ashmem: initialized
[    1.524185] mmc_host mmc0: Bus speed (slot 0) = 25000000Hz (slot req 25000000Hz, actual 25000000HZ div = 0)
[    1.533239] usb 4-1: new low-speed USB device number 2 using ohci-platform
[    1.543530] mmc0: new SDHC card at address 0001
[    1.559034] NET: Registered protocol family 10
[    1.570967] mmcblk0: mmc0:0001 00000 14.9 GiB 
[    1.580455] Segment Routing with IPv6
[    1.600779] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    1.613389] NET: Registered protocol family 17
[    1.617091]  mmcblk0: p1 p2 p3 p4 p5
[    1.625256] Key type dns_resolver registered
[    1.647105] ThumbEE CPU extension supported.
[    1.658241] Registering SWP/SWPB emulation handler
[    1.670459] Loading compiled-in X.509 certificates
[    1.702062] hctosys: unable to open rtc device (rtc0)
[    1.716639] EXT4-fs (mmcblk0p5): mounting ext2 file system using the ext4 subsystem
[    1.779007] EXT4-fs (mmcblk0p5): warning: mounting unchecked fs, running e2fsck is recommended
[    1.795755] EXT4-fs (mmcblk0p5): mounted filesystem without journal. Opts: (null)
[    1.807395] VFS: Mounted root (ext2 filesystem) on device 179:5.
[    1.867392] devtmpfs: mounted
[    1.882870] Freeing unused kernel memory: 1024K
[    1.894318] Run /sbin/init as init process
[    1.895243] input:   USB Keyboard as /devices/platform/300a0000.usb/usb4/4-1/4-1:1.0/0003:04D9:1603.0001/input/input0
[    1.984084] hid-generic 0003:04D9:1603.0001: input: USB HID v1.10 Keyboard [  USB Keyboard] on usb-300a0000.usb-1/input0
[    2.018216] input:   USB Keyboard System Control as /devices/platform/300a0000.usb/usb4/4-1/4-1:1.1/0003:04D9:1603.0002/input/input1
[    2.093913] input:   USB Keyboard Consumer Control as /devices/platform/300a0000.usb/usb4/4-1/4-1:1.1/0003:04D9:1603.0002/input/input2
[    2.107256] hid-generic 0003:04D9:1603.0002: input: USB HID v1.10 Device [  USB Keyboard] on usb-300a0000.usb-1/input1
[    4.553334] udevd[195]: starting version 3.2.2
[    4.672363] random: udevd: uninitialized urandom read (16 bytes read)
[    4.691014] random: udevd: uninitialized urandom read (16 bytes read)
[    4.704008] random: udevd: uninitialized urandom read (16 bytes read)
[    4.919393] udevd[196]: starting eudev-3.2.2
[    9.083660] EXT4-fs (mmcblk0p5): re-mounted. Opts: (null)
[   12.189860] urandom_read: 2 callbacks suppressed
[   12.189888] random: dd: uninitialized urandom read (512 bytes read)
[   16.028568] random: sshd: uninitialized urandom read (32 bytes read)
[   16.358690] ttyS2 - failed to request DMA

workstation login: 
```
