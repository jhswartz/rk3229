# eMMC Installation


### Prerequisites

- An RK3229 based device
- Physical access to the appropriate UART and eMMC
- A serial communications program for UART access
- Compile [rkdeveloptool](https://github.com/rockchip-linux/rkdeveloptool)
- Merge the RK322XATMINIALL loader from [rkbin](https://github.com/rockchip-linux/rkbin)
- [Compile OP-TEE, U-Boot and Linux for RK3229](COMPILE.md)


### Enter MASKROM mode

##### Option 1
Momentarily disable the eMMC by shorting clock to ground upon boot.

##### Option 2
Boot into the proprietary loader's ROCKUSB mode, and then switch to MASKROM mode.
```
# rkdeveloptool rd 3
```


### Push the proprietary loader for eMMC access via USB
```
# rkdeveloptool db rk322x_loader_at_v1.08.256.bin 
```


### Backup the existing eMMC loader
```
# rkdeveloptool rl 0x0000 0x4000 original-loader.img
```
*This would be a good time to backup the rest of your eMMC!*


### Erase the existing eMMC loader
```
# dd if=/dev/zero of=zero-loader.img bs=512 count=$((0x4000))
# rkdeveloptool wl 0x0000 zero-loader.img
```


### Write U-Boot to the eMMC
```
# rkdeveloptool wl 0x0040 idbloader.img
# rkdeveloptool wl 0x4000 u-boot.itb 
```


### Reboot the device
```
# rkdeveloptool rd
```


### Connect to UART2
```
# dterm /dev/ttyUSB0 1500000 8 n 1
TPL InitReturning to boot ROM...

U-Boot SPL 2019.07-rc4-dirty (Jun 25 2019 - 10:28:39 +0000)
Trying to boot from MMC1
D/TC:0 0 add_phys_mem:563 VCORE_UNPG_RX_PA type TEE_RAM_RX 0x68400000 size 0x0004f000
D/TC:0 0 add_phys_mem:563 VCORE_UNPG_RW_PA type TEE_RAM_RW 0x6844f000 size 0x000b1000
D/TC:0 0 add_phys_mem:563 TA_RAM_START type TA_RAM 0x68500000 size 0x00100000
D/TC:0 0 add_phys_mem:563 TEE_SHMEM_START type NSEC_SHM 0x68600000 size 0x00100000
D/TC:0 0 add_phys_mem:563 ROUNDDOWN(0x10100000, CORE_MMU_PGDIR_SIZE) type IO_SEC 0x10100000 size 0x22000000
D/TC:0 0 add_phys_mem:563 ROUNDDOWN(0x10080000, CORE_MMU_PGDIR_SIZE) type IO_NSEC 0x10000000 size 0x00100000
D/TC:0 0 verify_special_mem_areas:501 No NSEC DDR memory area defined
D/TC:0 0 add_va_space:602 type RES_VASPACE size 0x00a00000
D/TC:0 0 add_va_space:602 type SHM_VASPACE size 0x02000000
D/TC:0 0 dump_mmap_table:738 type TEE_RAM_RX   va 0x68400000..0x6844efff pa 0x68400000..0x6844efff size 0x0004f000 (smallpg)
D/TC:0 0 dump_mmap_table:738 type TEE_RAM_RW   va 0x6844f000..0x684fffff pa 0x6844f000..0x684fffff size 0x000b1000 (smallpg)
D/TC:0 0 dump_mmap_table:738 type SHM_VASPACE  va 0x68500000..0x6a4fffff pa 0x00000000..0x01ffffff size 0x02000000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type IO_SEC       va 0x6a500000..0x8c4fffff pa 0x10100000..0x320fffff size 0x22000000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type RES_VASPACE  va 0x8c500000..0x8cefffff pa 0x00000000..0x009fffff size 0x00a00000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type TA_RAM       va 0x8cf00000..0x8cffffff pa 0x68500000..0x685fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type NSEC_SHM     va 0x8d000000..0x8d0fffff pa 0x68600000..0x686fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type IO_NSEC      va 0x8d100000..0x8d1fffff pa 0x10000000..0x100fffff size 0x00100000 (pgdir)
D/TC:0 0 core_mmu_alloc_l2:274 L2 table used: 1/4
I/TC: 
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[0] with top at 0x68476838
D/TC:0 0 init_canaries:172 watch *0x6847683c
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[1] with top at 0x68476f78
D/TC:0 0 init_canaries:172 watch *0x68476f7c
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[2] with top at 0x684776b8
D/TC:0 0 init_canaries:172 watch *0x684776bc
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[3] with top at 0x68477df8
D/TC:0 0 init_canaries:172 watch *0x68477dfc
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[0] with top at 0x68478638
D/TC:0 0 init_canaries:173 watch *0x6847863c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[1] with top at 0x68478e78
D/TC:0 0 init_canaries:173 watch *0x68478e7c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[2] with top at 0x684796b8
D/TC:0 0 init_canaries:173 watch *0x684796bc
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[3] with top at 0x68479ef8
D/TC:0 0 init_canaries:173 watch *0x68479efc
D/TC:0 0 init_canaries:175 #Stack canaries for stack_thread[0] with top at 0x6847bf38
D/TC:0 0 init_canaries:175 watch *0x6847bf3c
D/TC:0 0 init_canaries:175 #Stack canaries for stack_thread[1] with top at 0x6847df78
D/TC:0 0 init_canaries:175 watch *0x6847df7c
I/TC: OP-TEE version: 3.5.0 #1 Fri Jun 21 08:51:43 UTC 2019 arm
D/TC:0 0 check_ta_store:536 TA store: "Secure Storage TA"
D/TC:0 0 check_ta_store:536 TA store: "REE [buffered]"
D/TC:0 0 mobj_mapped_shm_init:721 Shared memory address range: 68500000, 6a500000
I/TC: Initialized
D/TC:0 0 init_primary_helper:1079 Primary CPU switching to normal world boot


U-Boot 2019.07-rc4-dirty (Jun 25 2019 - 10:28:55 +0000)

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
Warning: ethernet@30200000 (eth0) using random MAC address - ce:b3:e0:44:17:9a
eth0: ethernet@30200000
Hit any key to stop autoboot:  2
```

Hit any key before the autoboot countdown elapses.


### Allow U-Boot to partition the eMMC
```
=> mmc list
dwmmc@30000000: 1
dwmmc@30020000: 0 (eMMC)
=> mmc dev 0
switch to partitions #0, OK
mmc0(part 0) is current device
=> mmc info
Device: dwmmc@30020000
Manufacturer ID: 88
OEM: 103
Name: NCard 
Bus Speed: 52000000
Mode: MMC High Speed (52MHz)
Rd Block Len: 512
MMC version 5.1
High Capacity: Yes
Capacity: 7.3 GiB
Bus Width: 8-bit
Erase Group Size: 512 KiB
HC WP Group Size: 4 MiB
User Capacity: 7.3 GiB WRREL
Boot Capacity: 4 MiB ENH
RPMB Capacity: 4 MiB ENH
=> gpt write mmc 0 $partitions
Writing GPT: success!
=> gpt verify mmc 0 $partitions
Verify GPT: success!
=> mmc part

Partition Map for MMC device 0  --   Partition Type: EFI

Part    Start LBA       End LBA         Name
        Attributes
        Type GUID
        Partition GUID
  1     0x00000040      0x00001f7f      "loader1"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   87e47153-c793-974d-b6c5-b40dc8caf1cf
  2     0x00004000      0x00005fff      "loader2"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   1f78251c-fed7-0c40-b611-ea9363326972
  3     0x00006000      0x00007fff      "trust"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   4dfb4e04-c1bf-1e4d-8e2f-e9cf51d418c4
  4     0x00008000      0x0003ffff      "boot"
        attrs:  0x0000000000000004
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   0c795b49-27e9-114c-8815-ca231b67e3eb
  5     0x00040000      0x00e8ffde      "rootfs"
        attrs:  0x0000000000000000
        type:   ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
        guid:   69dad710-2ce4-4e3c-b16c-21a1d49abed3
```


### Expose the eMMC as a USB Mass Storage Device

```
=> ums 0 mmc 0
UMS: LUN 0, dev 0, hwpart 0, sector 0x0, count 0xe90000
/
^]
dterm> quit
# tail -n 20 /var/log/messages | grep -E 'usb |sd '
Jun 25 14:15:41 workstation kernel: [33190640.560866] usb 5-5.3: SerialNumber: rockchip
Jun 25 14:25:14 workstation kernel: [33191213.694014] usb 5-5.3: USB disconnect, device number 85
Jun 25 14:43:44 workstation kernel: [33192324.164079] usb 5-5.3: new high-speed USB device number 86 using ehci-pci
Jun 25 14:43:45 workstation kernel: [33192324.272700] usb 5-5.3: New USB device found, idVendor=2207, idProduct=320a
Jun 25 14:43:45 workstation kernel: [33192324.272705] usb 5-5.3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
Jun 25 14:43:45 workstation kernel: [33192324.272708] usb 5-5.3: Product: USB download gadget
Jun 25 14:43:45 workstation kernel: [33192324.272710] usb 5-5.3: Manufacturer: Rockchip
Jun 25 14:43:46 workstation kernel: [33192325.273181] sd 412:0:0:0: Attached scsi generic sg4 type 0
Jun 25 14:43:46 workstation kernel: [33192325.274382] sd 412:0:0:0: [sdd] 15269888 512-byte logical blocks: (7.81 GB/7.28 GiB)
Jun 25 14:43:46 workstation kernel: [33192325.274711] sd 412:0:0:0: [sdd] Write Protect is off
Jun 25 14:43:46 workstation kernel: [33192325.275082] sd 412:0:0:0: [sdd] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
Jun 25 14:43:46 workstation kernel: [33192325.300110] sd 412:0:0:0: [sdd] Attached SCSI removable disk
# lsblk /dev/sdd
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdd      8:48   1  7.3G  0 disk 
├─sdd1   8:49   1  3.9M  0 part 
├─sdd2   8:50   1    4M  0 part 
├─sdd3   8:51   1    4M  0 part 
├─sdd4   8:52   1  112M  0 part 
└─sdd5   8:53   1  7.2G  0 part 
```


### Initialize the boot filesystem

```
# cd $BUILD
# mkfs.vfat /dev/sdd4
# BOOT=/mnt/sdd4
# mkdir $BOOT
# mount /dev/sdd4 $BOOT
# mkdir $BOOT/dtb
# cp linux/arch/arm/boot/dts/rk3229-xms6.dtb $BOOT/dtb
# cp linux/arch/arm/boot/zImage $BOOT
# mkdir $BOOT/extlinux
# cat > $BOOT/extlinux/extlinux.conf
timeout 5
label default
        fdt /dtb/rk3229-xms6.dtb
        linux /zImage
	append console=tty0 console=ttyS2,1500000n8 drm.debug=0x0e drm.edid_firmware=edid/1920x1080.bin video=HDMI-A-1:e ignore_loglevel debug rw root=/dev/mmcblk1p5 rootfstype=ext2 rootwait
^D
# umount $BOOT
```


### Initialize the root filesystem

```
# cd $BUILD
# mkfs.ext2 /dev/sdd5
# ROOTFS=/mnt/sdd5
# mkdir $ROOTFS
# mount /dev/sdd5 $ROOTFS
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
/dev/mmcblk1p4  /boot                   vfat            defaults        0       0
none            /tmp                    tmpfs           defaults        0       0
none            /sys/kernel/debug       debugfs         defaults        0       0
^D
# exit
```


### Copy the kernel modules to the root filesystem

```
# cd $BUILD/linux
# build build-modules_install.log -j2 INSTALL_MOD_PATH=$ROOTFS modules_install
# umount $ROOTFS
```


### Connect to UART2

```
# dterm /dev/ttyUSB0 1500000 8 n 1
^C
CTRL+C - Operation aborted
=> reset
resetting ...
TPL InitReturning to boot ROM...

U-Boot SPL 2019.07-rc4-dirty (Jun 25 2019 - 10:28:39 +0000)
Trying to boot from MMC1
D/TC:0 0 add_phys_mem:563 VCORE_UNPG_RX_PA type TEE_RAM_RX 0x68400000 size 0x0004f000
D/TC:0 0 add_phys_mem:563 VCORE_UNPG_RW_PA type TEE_RAM_RW 0x6844f000 size 0x000b1000
D/TC:0 0 add_phys_mem:563 TA_RAM_START type TA_RAM 0x68500000 size 0x00100000
D/TC:0 0 add_phys_mem:563 TEE_SHMEM_START type NSEC_SHM 0x68600000 size 0x00100000
D/TC:0 0 add_phys_mem:563 ROUNDDOWN(0x10100000, CORE_MMU_PGDIR_SIZE) type IO_SEC 0x10100000 size 0x22000000
D/TC:0 0 add_phys_mem:563 ROUNDDOWN(0x10080000, CORE_MMU_PGDIR_SIZE) type IO_NSEC 0x10000000 size 0x00100000
D/TC:0 0 verify_special_mem_areas:501 No NSEC DDR memory area defined
D/TC:0 0 add_va_space:602 type RES_VASPACE size 0x00a00000
D/TC:0 0 add_va_space:602 type SHM_VASPACE size 0x02000000
D/TC:0 0 dump_mmap_table:738 type TEE_RAM_RX   va 0x68400000..0x6844efff pa 0x68400000..0x6844efff size 0x0004f000 (smallpg)
D/TC:0 0 dump_mmap_table:738 type TEE_RAM_RW   va 0x6844f000..0x684fffff pa 0x6844f000..0x684fffff size 0x000b1000 (smallpg)
D/TC:0 0 dump_mmap_table:738 type SHM_VASPACE  va 0x68500000..0x6a4fffff pa 0x00000000..0x01ffffff size 0x02000000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type IO_SEC       va 0x6a500000..0x8c4fffff pa 0x10100000..0x320fffff size 0x22000000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type RES_VASPACE  va 0x8c500000..0x8cefffff pa 0x00000000..0x009fffff size 0x00a00000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type TA_RAM       va 0x8cf00000..0x8cffffff pa 0x68500000..0x685fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type NSEC_SHM     va 0x8d000000..0x8d0fffff pa 0x68600000..0x686fffff size 0x00100000 (pgdir)
D/TC:0 0 dump_mmap_table:738 type IO_NSEC      va 0x8d100000..0x8d1fffff pa 0x10000000..0x100fffff size 0x00100000 (pgdir)
D/TC:0 0 core_mmu_alloc_l2:274 L2 table used: 1/4
I/TC: 
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[0] with top at 0x68476838
D/TC:0 0 init_canaries:172 watch *0x6847683c
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[1] with top at 0x68476f78
D/TC:0 0 init_canaries:172 watch *0x68476f7c
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[2] with top at 0x684776b8
D/TC:0 0 init_canaries:172 watch *0x684776bc
D/TC:0 0 init_canaries:172 #Stack canaries for stack_tmp[3] with top at 0x68477df8
D/TC:0 0 init_canaries:172 watch *0x68477dfc
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[0] with top at 0x68478638
D/TC:0 0 init_canaries:173 watch *0x6847863c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[1] with top at 0x68478e78
D/TC:0 0 init_canaries:173 watch *0x68478e7c
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[2] with top at 0x684796b8
D/TC:0 0 init_canaries:173 watch *0x684796bc
D/TC:0 0 init_canaries:173 #Stack canaries for stack_abt[3] with top at 0x68479ef8
D/TC:0 0 init_canaries:173 watch *0x68479efc
D/TC:0 0 init_canaries:175 #Stack canaries for stack_thread[0] with top at 0x6847bf38
D/TC:0 0 init_canaries:175 watch *0x6847bf3c
D/TC:0 0 init_canaries:175 #Stack canaries for stack_thread[1] with top at 0x6847df78
D/TC:0 0 init_canaries:175 watch *0x6847df7c
I/TC: OP-TEE version: 3.5.0 #1 Fri Jun 21 08:51:43 UTC 2019 arm
D/TC:0 0 check_ta_store:536 TA store: "Secure Storage TA"
D/TC:0 0 check_ta_store:536 TA store: "REE [buffered]"
D/TC:0 0 mobj_mapped_shm_init:721 Shared memory address range: 68500000, 6a500000
I/TC: Initialized
D/TC:0 0 init_primary_helper:1079 Primary CPU switching to normal world boot


U-Boot 2019.07-rc4-dirty (Jun 25 2019 - 10:28:55 +0000)

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
Warning: ethernet@30200000 (eth0) using random MAC address - be:47:59:6d:f1:d7
eth0: ethernet@30200000
Hit any key to stop autoboot:  2  1  0 
switch to partitions #0, OK
mmc0(part 0) is current device
Scanning mmc 0:4...
Found /extlinux/extlinux.conf
Retrieving file: /extlinux/extlinux.conf
353 bytes read in 2 ms (171.9 KiB/s)
1:      default
Retrieving file: /zImage
5520760 bytes read in 204 ms (25.8 MiB/s)
append: console=tty0 console=ttyS2,1500000n8 video=HDMI-A-1:1920x1080@60 ignore_loglevel debug rw root=/dev/mmcblk1p5 rootfstype=ext2 rootwait
Retrieving file: /dtb/rk3229-xms6.dtb
23569 bytes read in 5 ms (4.5 MiB/s)
## Flattened Device Tree blob at 61f00000
   Booting using the fdt blob at 0x61f00000
   Loading Device Tree to 683f7000, end 683ffc10 ... OK

Starting kernel ...

D/TC:0   psci_cpu_on:278 core_id: 1
D/TC:1   init_secondary_helper:1103 Secondary CPU Switching to normal world boot
D/TC:0   psci_cpu_on:278 core_id: 2
D/TC:2   init_secondary_helper:1103 Secondary CPU Switching to normal world boot
D/TC:0   psci_cpu_on:278 core_id: 3
D/TC:3   init_secondary_helper:1103 Secondary CPU Switching to normal world boot
[    0.000000] Booting Linux on physical CPU 0xf00
[    0.000000] Linux version 5.4.0 (justin@v01.28459.vpscontrol.net) (gcc version 6.3.0 20170516 (Debian 6.3.0-18)) #1 SMP Mon Nov 25 21:43:34 UTC 2019
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Mecer Xtreme Mini S6
[    0.000000] printk: debug: ignoring loglevel setting.
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] cma: Reserved 64 MiB at 0x9c000000
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
[    0.000000] percpu: Embedded 19 pages/cpu s47820 r8192 d21812 u77824
[    0.000000] pcpu-alloc: s47820 r8192 d21812 u77824 alloc=19*4096
[    0.000000] pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 260096
[    0.000000] Kernel command line: console=tty0 console=ttyS2,1500000n8 video=HDMI-A-1:1920x1080@60 ignore_loglevel debug rw root=/dev/mmcblk1p5 rootfstype=ext2 rootwait
[    0.000000] Dentry cache hash table entries: 131072 (order: 7, 524288 bytes, linear)
[    0.000000] Inode-cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 958456K/1046528K available (8192K kernel code, 557K rwdata, 2188K rodata, 1024K init, 282K bss, 22536K reserved, 65536K cma-reserved, 196608K highmem)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=16 to nr_cpu_ids=4.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=4
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000000] random: get_random_bytes called from start_kernel+0x324/0x4bc with crng_init=0
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x588fe9dc0, max_idle_ns: 440795202592 ns
[    0.000008] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398046511097ns
[    0.000022] Switching to timer-based delay loop, resolution 41ns
[    0.000996] Console: colour dummy device 80x30
[    0.001472] printk: console [tty0] enabled
[    0.001536] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=240000)
[    0.001570] pid_max: default: 32768 minimum: 301
[    0.001758] Mount-cache hash table entries: 2048 (order: 1, 8192 bytes, linear)
[    0.001797] Mountpoint-cache hash table entries: 2048 (order: 1, 8192 bytes, linear)
[    0.001919] *** VALIDATE tmpfs ***
[    0.002633] *** VALIDATE proc ***
[    0.002817] *** VALIDATE cgroup1 ***
[    0.002844] *** VALIDATE cgroup2 ***
[    0.002869] CPU: Testing write buffer coherency: ok
[    0.003381] /cpus/cpu@f00 missing clock-frequency property
[    0.003430] /cpus/cpu@f01 missing clock-frequency property
[    0.003459] /cpus/cpu@f02 missing clock-frequency property
[    0.003488] /cpus/cpu@f03 missing clock-frequency property
[    0.003511] CPU0: thread -1, cpu 0, socket 15, mpidr 80000f00
[    0.004265] Setting up static identity map for 0x60100000 - 0x60100060
[    0.004483] rcu: Hierarchical SRCU implementation.
[    0.008131] smp: Bringing up secondary CPUs ...
[    0.010843] CPU1: thread -1, cpu 1, socket 15, mpidr 80000f01
[    0.013605] CPU2: thread -1, cpu 2, socket 15, mpidr 80000f02
[    0.016232] CPU3: thread -1, cpu 3, socket 15, mpidr 80000f03
[    0.016380] smp: Brought up 1 node, 4 CPUs
[    0.016456] SMP: Total of 4 processors activated (192.00 BogoMIPS).
[    0.016474] CPU: All CPU(s) started in SVC mode.
[    0.017363] devtmpfs: initialized
[    0.023794] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 5
[    0.024215] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.024270] futex hash table entries: 1024 (order: 4, 65536 bytes, linear)
[    0.027134] pinctrl core: initialized pinctrl subsystem
[    0.028769] NET: Registered protocol family 16
[    0.031061] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.033184] cpuidle: using governor menu
[    0.033736] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.033770] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.034431] Serial: AMBA PL011 UART driver
[    0.068643] vcc_sys: supplied by dc_12v
[    0.069084] vccio_1v8: supplied by vcc_sys
[    0.069362] vcc_host: supplied by vcc_sys
[    0.069604] vccio_3v3: supplied by vcc_sys
[    0.069837] vcc_phy: supplied by vccio_1v8
[    0.070317] iommu: Default domain type: Translated 
[    0.071845] SCSI subsystem initialized
[    0.072201] usbcore: registered new interface driver usbfs
[    0.072280] usbcore: registered new interface driver hub
[    0.072442] usbcore: registered new device driver usb
[    0.072701] pps_core: LinuxPPS API ver. 1 registered
[    0.072724] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.072789] EDAC MC: Ver: 3.0.0
[    0.075365] clocksource: Switched to clocksource arch_sys_counter
[    0.124776] *** VALIDATE ramfs ***
[    0.135748] thermal_sys: Registered thermal governor 'step_wise'
[    0.136400] NET: Registered protocol family 2
[    0.137335] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 6144 bytes, linear)
[    0.137402] TCP established hash table entries: 8192 (order: 3, 32768 bytes, linear)
[    0.137516] TCP bind hash table entries: 8192 (order: 4, 65536 bytes, linear)
[    0.137669] TCP: Hash tables configured (established 8192 bind 8192)
[    0.137882] UDP hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.137970] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes, linear)
[    0.138275] NET: Registered protocol family 1
[    0.138345] PCI: CLS 0 bytes, default 64
[    0.139612] hw perfevents: enabled with armv7_cortex_a7 PMU driver, 5 counters available
[    0.141718] Initialise system trusted keyrings
[    0.142122] workingset: timestamp_bits=30 max_order=18 bucket_order=0
[    0.149981] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.150248] ntfs: driver 2.1.32 [Flags: R/O].
[    0.151021] Key type asymmetric registered
[    0.151053] Asymmetric key parser 'x509' registered
[    0.151138] bounce: pool size: 64 pages
[    0.151209] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)
[    0.151237] io scheduler mq-deadline registered
[    0.151253] io scheduler kyber registered
[    0.152687] inno-hdmi-phy 12030000.hdmi-phy: IRQ index 0 not found
[    0.159313] pwm-regulator: supplied by vcc_sys
[    0.159915] pwm-regulator: supplied by vcc_sys
[    0.228266] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.231420] printk: console [ttyS2] disabled
[    0.231548] 11030000.serial: ttyS2 at MMIO 0x11030000 (irq = 28, base_baud = 1500000) is a 16550A
[    0.292882] printk: console [ttyS2] enabled
[    0.295200] rockchip-vop 20050000.vop: Adding to iommu group 0
[    0.299250] rockchip-drm display-subsystem: bound 20050000.vop (ops 0xc0953350)
[    0.300207] dwhdmi-rockchip 200a0000.hdmi: Detected HDMI TX controller v2.01a with HDCP (inno_dw_hdmi_phy2)
[    0.301732] dwhdmi-rockchip 200a0000.hdmi: registered DesignWare HDMI I2C bus driver
[    0.302829] rockchip-drm display-subsystem: bound 200a0000.hdmi (ops 0xc0956464)
[    0.303510] [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
[    0.304101] [drm] No driver support for vblank timestamp query.
[    0.438752] Console: switching to colour frame buffer device 240x67
[    0.489572] rockchip-drm display-subsystem: fb0: rockchipdrmfb frame buffer device
[    0.491513] [drm] Initialized rockchip 1.0.0 20140818 for display-subsystem on minor 0
[    0.495054] lima 20000000.gpu: IRQ pmu not found
[    0.496015] lima 20000000.gpu: IRQ ppmmu2 not found
[    0.496635] lima 20000000.gpu: IRQ ppmmu3 not found
[    0.497285] lima 20000000.gpu: gp - mali400 version major 1 minor 1
[    0.498221] lima 20000000.gpu: pp0 - mali400 version major 1 minor 1
[    0.499078] lima 20000000.gpu: pp1 - mali400 version major 1 minor 1
[    0.499860] lima 20000000.gpu: IRQ pp2 not found
[    0.500426] lima 20000000.gpu: IRQ pp3 not found
[    0.500993] lima 20000000.gpu: l2 cache 64K, 4-way, 64byte cache line, 64bit external bus
[    0.503167] lima 20000000.gpu: bus rate = 297000000
[    0.503836] lima 20000000.gpu: mod rate = 297000000
[    0.505128] [drm] Initialized lima 1.0.0 20190217 for 20000000.gpu on minor 1
[    0.524385] brd: module loaded
[    0.543196] loop: module loaded
[    0.545801] libphy: Fixed MDIO Bus: probed
[    0.547146] rk_gmac-dwmac 30200000.ethernet: IRQ eth_wake_irq not found
[    0.547984] rk_gmac-dwmac 30200000.ethernet: IRQ eth_lpi not found
[    0.548948] rk_gmac-dwmac 30200000.ethernet: PTP uses main clock
[    0.549875] rk_gmac-dwmac 30200000.ethernet: clock input or output? (output).
[    0.550749] rk_gmac-dwmac 30200000.ethernet: Can not read property: tx_delay.
[    0.551611] rk_gmac-dwmac 30200000.ethernet: set tx_delay to 0x30
[    0.552347] rk_gmac-dwmac 30200000.ethernet: Can not read property: rx_delay.
[    0.553199] rk_gmac-dwmac 30200000.ethernet: set rx_delay to 0x10
[    0.553950] rk_gmac-dwmac 30200000.ethernet: integrated PHY? (yes).
[    0.554817] rk_gmac-dwmac 30200000.ethernet: cannot get clock clk_mac_speed
[    0.561506] rk_gmac-dwmac 30200000.ethernet: init for RMII
[    0.605851] rk_gmac-dwmac 30200000.ethernet: User ID: 0x10, Synopsys ID: 0x35
[    0.606788] rk_gmac-dwmac 30200000.ethernet:         DWMAC1000
[    0.607438] rk_gmac-dwmac 30200000.ethernet: DMA HW capability register supported
[    0.608339] rk_gmac-dwmac 30200000.ethernet: RX Checksum Offload Engine supported
[    0.609235] rk_gmac-dwmac 30200000.ethernet: COE Type 2
[    0.609867] rk_gmac-dwmac 30200000.ethernet: TX Checksum insertion supported
[    0.610707] rk_gmac-dwmac 30200000.ethernet: Wake-Up On Lan supported
[    0.611544] rk_gmac-dwmac 30200000.ethernet: Normal descriptors
[    0.612264] rk_gmac-dwmac 30200000.ethernet: Ring mode enabled
[    0.622488] rk_gmac-dwmac 30200000.ethernet: Enable RX Mitigation via HW Watchdog Timer
[    0.632859] rk_gmac-dwmac 30200000.ethernet: device MAC address 16:74:5b:0b:09:9a
[    0.643569] libphy: stmmac: probed
[    0.656857] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.667467] ehci-pci: EHCI PCI platform driver
[    0.677921] ehci-platform: EHCI generic platform driver
[    0.690784] ehci-platform 30080000.usb: EHCI Host Controller
[    0.701101] ehci-platform 30080000.usb: new USB bus registered, assigned bus number 1
[    0.712148] ehci-platform 30080000.usb: irq 39, io mem 0x30080000
[    0.745379] ehci-platform 30080000.usb: USB 2.0 started, EHCI 1.00
[    0.757403] hub 1-0:1.0: USB hub found
[    0.767851] hub 1-0:1.0: 1 port detected
[    0.781219] ehci-platform 300c0000.usb: EHCI Host Controller
[    0.791573] ehci-platform 300c0000.usb: new USB bus registered, assigned bus number 2
[    0.802755] ehci-platform 300c0000.usb: irq 41, io mem 0x300c0000
[    0.835391] ehci-platform 300c0000.usb: USB 2.0 started, EHCI 1.00
[    0.847187] hub 2-0:1.0: USB hub found
[    0.857464] hub 2-0:1.0: 1 port detected
[    0.870694] ehci-platform 30100000.usb: EHCI Host Controller
[    0.880843] ehci-platform 30100000.usb: new USB bus registered, assigned bus number 3
[    0.892915] ehci-platform 30100000.usb: irq 43, io mem 0x30100000
[    0.925401] ehci-platform 30100000.usb: USB 2.0 started, EHCI 1.00
[    0.936816] hub 3-0:1.0: USB hub found
[    0.946493] hub 3-0:1.0: 1 port detected
[    0.956903] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.966504] ohci-platform: OHCI generic platform driver
[    0.976415] ohci-platform 300a0000.usb: Generic Platform OHCI controller
[    0.985854] ohci-platform 300a0000.usb: new USB bus registered, assigned bus number 4
[    0.996044] ohci-platform 300a0000.usb: irq 40, io mem 0x300a0000
[    1.070923] hub 4-0:1.0: USB hub found
[    1.080360] hub 4-0:1.0: 1 port detected
[    1.090853] ohci-platform 300e0000.usb: Generic Platform OHCI controller
[    1.100090] ohci-platform 300e0000.usb: new USB bus registered, assigned bus number 5
[    1.110117] ohci-platform 300e0000.usb: irq 42, io mem 0x300e0000
[    1.190846] hub 5-0:1.0: USB hub found
[    1.200507] hub 5-0:1.0: 1 port detected
[    1.210794] ohci-platform 30120000.usb: Generic Platform OHCI controller
[    1.220375] ohci-platform 30120000.usb: new USB bus registered, assigned bus number 6
[    1.230739] ohci-platform 30120000.usb: irq 44, io mem 0x30120000
[    1.310854] hub 6-0:1.0: USB hub found
[    1.320713] hub 6-0:1.0: 1 port detected
[    1.331717] usbcore: registered new interface driver usb-storage
[    1.342016] i2c /dev entries driver
[    1.354783] rockchip-thermal 11150000.tsadc: Missing tshut-polarity property, using default (low)
[    1.364893] rockchip-thermal 11150000.tsadc: Missing rockchip,grf property
[    1.376615] softdog: initialized. soft_noboot=0 soft_margin=60 sec soft_panic=0 (nowayout=0)
[    1.387325] Synopsys Designware Multimedia Card Interface Driver
[    1.398434] dwmmc_rockchip 30000000.dwmmc: IDMAC supports 32-bit address mode.
[    1.408949] dwmmc_rockchip 30000000.dwmmc: Using internal DMA controller.
[    1.419031] dwmmc_rockchip 30000000.dwmmc: Version ID is 270a
[    1.429126] dwmmc_rockchip 30000000.dwmmc: DW MMC controller at irq 36,32 bit host data width,256 deep fifo
[    1.452889] mmc_host mmc0: Bus speed (slot 0) = 400000Hz (slot req 400000Hz, actual 400000HZ div = 0)
[    1.477467] dwmmc_rockchip 30020000.dwmmc: IDMAC supports 32-bit address mode.
[    1.488342] dwmmc_rockchip 30020000.dwmmc: Using internal DMA controller.
[    1.498697] dwmmc_rockchip 30020000.dwmmc: Version ID is 270a
[    1.509203] dwmmc_rockchip 30020000.dwmmc: DW MMC controller at irq 37,32 bit host data width,256 deep fifo
[    1.520287] mmc_host mmc1: card is non-removable.
[    1.539955] random: fast init done
[    1.543835] mmc_host mmc1: Bus speed (slot 0) = 1160156Hz (slot req 400000Hz, actual 290039HZ div = 2)
[    1.575895] ledtrig-cpu: registered to indicate activity on CPUs
[    1.587480] usbcore: registered new interface driver usbhid
[    1.598479] usbhid: USB HID core driver
[    1.610114] ashmem: initialized
[    1.624381] NET: Registered protocol family 10
[    1.637384] Segment Routing with IPv6
[    1.648513] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    1.661052] NET: Registered protocol family 17
[    1.672526] Key type dns_resolver registered
[    1.683835] ThumbEE CPU extension supported.
[    1.694827] Registering SWP/SWPB emulation handler
[    1.706414] Loading compiled-in X.509 certificates
[    1.747013] hctosys: unable to open rtc device (rtc0)
[    1.760130] ttyS2 - failed to request DMA
[    1.771898] Waiting for root device /dev/mmcblk1p5...
[    1.808914] mmc_host mmc1: Bus speed (slot 0) = 37125000Hz (slot req 37500000Hz, actual 37125000HZ div = 0)
[    1.821760] mmc1: new high speed MMC card at address 0001
[    1.835010] mmcblk1: mmc1:0001 NCard  7.28 GiB 
[    1.847563] mmcblk1boot0: mmc1:0001 NCard  partition 1 4.00 MiB
[    1.860268] mmcblk1boot1: mmc1:0001 NCard  partition 2 4.00 MiB
[    1.872193] mmcblk1rpmb: mmc1:0001 NCard  partition 3 4.00 MiB, chardev (246:0)
[    1.894974]  mmcblk1: p1 p2 p3 p4 p5
[    1.937452] EXT4-fs (mmcblk1p5): mounting ext2 file system using the ext4 subsystem
[    1.957495] EXT4-fs (mmcblk1p5): warning: mounting unchecked fs, running e2fsck is recommended
[    1.972458] EXT4-fs (mmcblk1p5): mounted filesystem without journal. Opts: (null)
[    1.984596] VFS: Mounted root (ext2 filesystem) on device 179:5.
[    2.003229] devtmpfs: mounted
[    2.017195] Freeing unused kernel memory: 1024K
[    2.029209] Run /sbin/init as init process
```
