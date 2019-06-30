# eMMC Partition Replacement


### Prerequisites

- An RK3229 based device, assumedly shipped with Android with all eMMC partitions still entact
- A USB-A Male cable to USB-A Male cable
- Compile [rkflashtool](https://github.com/linux-rockchip/rkflashtool)
- [Compile OP-TEE, U-Boot and Linux for RK3229](COMPILE.md)


### Initialize the environment

```
$ export SCRIPTS=`readlink -f $BUILD/scripts`
$ export REPLACEMENT=`readlink -f $BUILD/replacement`
$ export ROOTFS=`readlink -f $REPLACEMENT/rootfs`
$ export INITRD=`readlink -f $REPLACEMENT/initrd`
$ export SYSTEM=`readlink -f $REPLACEMENT/system`
$ mkdir $REPLACEMENT
$ mkdir $ROOTFS
$ mkdir $INITRD
$ mkdir $SYSTEM
```


### Enter loader (rockusb) mode

Insert one end of the USB-A Male to USB-A Male cable into your host computer, then press and hold the device's Loader button before inserting the other end of the cable into the device's Download USB port. Release the Loader button after a few seconds.


### Enumerate eMMC partitions

Rockchip's branches of U-boot and the Linux kernel rely on a comma separated list of partition declarations, stored in the value of the **mtdparts** kernel command line parameter, to determine partition boundaries instead of using an on-disk partition table.

Each declaration in the partition list is formatted as **size**@**offset**(***name***) where **size** and **offset** are specified in sectors, and a size of **-** indicates the use of all subsequent sectors until the end of the disk.

```
# rkflashtool p > $REPLACEMENT/parameters
# sed -e '/^CMDLINE:/!d' -e 's/^CMDLINE:.*mtdparts=rk29xxnand:\(.*\)$/\1/g' -e 's/,/\n/g' $REPLACEMENT/parameters
0x00002000@0x00002000(uboot)
0x00004000@0x00004000(trust)
0x00002000@0x00008000(misc)
0x00000800@0x0000A000(baseparamer)
0x00007800@0x0000A800(resource)
0x00006000@0x00012000(kernel)
0x0000C000@0x00018000(boot)
0x00010000@0x00024000(recovery)
0x00020000@0x00034000(backup)
0x00040000@0x00054000(cache)
0x00008000@0x00094000(metadata)
0x00002000@0x0009C000(kpanic)
0x00400000@0x0009E000(system)
-@0x0049E000(userdata)
```


### Prepare the kernel command line

```
# sed -i -e 's|^\(CMDLINE:\).*\(mtdparts=rk29xxnand:.*\)$|\1console=ttyS2,1500000n8 console=tty0 debug video=HDMI-A-1:1920x1080@60 \2|g' $REPLACEMENT/parameters
```


### Prepare contents of the root filesystem

The following commands bootstrap the current stable release of Devuan, if you wish to use another distribution copy it to $ROOTFS instead.

```
# qemu-debootstrap --arch armhf stable $ROOTFS http://pkgmaster.devuan.org/merged
# chroot $ROOTFS /bin/bash
# > /etc/motd
# > /etc/issue
# > /etc/issue.net
# echo "workstation" > /etc/hostname
# echo "T2:23:respawn:/sbin/getty -L ttyS2 1500000 linux" >> /etc/inittab
# cat > /etc/fstab << EOF
none            /tmp                    tmpfs           defaults        0       0
none            /sys/kernel/debug       debugfs         defaults        0       0
EOF
# apt-get update 
# apt-get install busybox 
# busybox --list-full > /tmp/applets
# exit
```


### Prepare the contents of the initrd

The libraries that you will need to copy may differ if you have opted to use a distribution other than Devuan for your root fileystem.

```
# cd $INITRD
# mkdir -p bin dev etc lib lib/arm-linux-gnueabihf mnt proc sbin tmp
# mknod -m 600 dev/console c 5 1
# mknod -m 600 dev/ttyS2 c 4 66
# cp $ROOTFS/lib/ld-linux-armhf.so.3 lib/
# cp $ROOTFS/lib/arm-linux-gnueabihf/{libc.so.6,libfdisk.so.1,libuuid.so.1,libblkid.so.1,libsmartcols.so.1,libtinfo.so.5} lib/arm-linux-gnueabihf/
# cp $ROOTFS/bin/busybox bin/
# cp $ROOTFS/sbin/{agetty,fdisk} sbin/
# for applet in `cat $ROOTFS/tmp/applets`; do mkdir -p `dirname $applet` && ln -s /bin/busybox $applet; done
# rm $ROOTFS/tmp/applets
# cat > etc/inittab << EOF
::sysinit:/bin/mount -t proc none /proc
::sysinit:/bin/mount -t devtmpfs none /dev
::sysinit:/bin/mount -t tmpfs none /tmp
::askfirst:-/bin/ash
ttyS2::respawn:/sbin/agetty -n -l /sbin/inituart ttyS2
EOF
# cat > sbin/inituart << EOF
#!/bin/sh
/bin/stty cs8 sane
exec /bin/ash
EOF
# cat > init << EOF
#!/bin/sh

mount -t proc none /proc
mount -t devtmpfs none /dev

counter=5

while [ $counter -gt 0 ]
do
        if [ -b /dev/mmcblk1p1 ]
        then
                break
        fi

        counter=$(($counter - 1))
        sleep 1
done

if [ $counter -gt 0 ]
then
        mount /dev/mmcblk1p1 /mnt
        exec chroot /mnt /sbin/init
else
        umount /dev
        umount /proc
        exec /sbin/init
fi
EOF
# chmod 755 sbin/{inituart,init}
# find . | cpio -o -H newc | gzip -9 > $BUILD/initrd.gz
```


### Pack the partition images

```
# $SCRIPTS/pack-resource $BUILD/linux/arch/arm/boot/dts/rk3229-xms6.dtb $REPLACEMENT/resource.img $((0x7800 * 512))
# $SCRIPTS/pack-kernel $BUILD/linux/arch/arm/boot/zImage $REPLACEMENT/kernel.img $((0x6000 * 512))
# $SCRIPTS/pack-kernel $BUILD/initrd.gz $REPLACEMENT/boot.img $((0xc000 * 512))
```


### Push the images and modified parameters

```
# rkflashtool w resource < $REPLACEMENT/resource.img
# rkflashtool w kernel < $REPLACEMENT/kernel.img
# rkflashtool w boot < $REPLACEMENT/boot.img
# rkflashtool P < $REPLACEMENT/parameters
```


### Reboot and access the HDMI or UART2 console

```
DDR Version V1.06 20171026
In
300MHz
DDR3
Bus Width=16 Col=11 Bank=8 Row=15 CS=1 Die Bus-Width=16 Size=1024MB
mach:2
OUT
Boot1 Release Time: 2017-06-12, version: 2.37
ChipType = 0xc, 296
SdmmcInit=2 0
BootCapSize=2000
UserCapSize=7456MB
FwPartOffset=2000 , 2000
SdmmcInit=0 2
StorageInit ok = 30253
SecureMode : SBOOT_MODE_NS
Code check OK! theLoader 0x60000000, 43051
Code check OK! theLoader 0x68400000, 54106
Enter Trust OS
INF TEE-CORE:init_primary_helper:319: Initializing (1.0.1-73-gf218b99 #3 Mon Nov 13 10:12:23 UTC 2017 arm)
INF TEE-CORE:init_primary_helper:320: Release version: 2.0
INF TEE-CORE:init_teecore:79: teecore inits done


U-Boot 2014.10-RK322X-06-02568-g69b34513b6-dirty (Jul 12 2018 - 15:15:42)

CPU: rk322x
cpu version = 3
CPU's clock information:
    arm pll = 600000000HZ
    periph pll = 600000000HZ
    ddr pll = 600000000HZ
    codec pll = 500000000HZ
Board:  Rockchip platform Board
Uboot as second level loader
DRAM:  Found dram banks: 1
Adding bank:0000000060000000(0000000040000000)
Reserve memory for trust os.
dram reserve bank: base = 0x68400000, size = 0x00100000
128 MiB
GIC CPU mask = 0x00000001
rk dma pl330 version: 1.4
remotectl v0.1
pwm freq=0x11e1a3
pwm_freq_nstime=0x355
SdmmcInit = 0 20
SdmmcInit = 2 0
storage init OK!
Using default environment

GetParam
Load FDT from resource image.
can't find dts node for fixed
No pmic detect.
Cannot find regulator pwm id
Cannot find regulator pwm id
SecureBootEn = 0, SecureBootLock = 0

#Boot ver: 2018-07-12#2.37
empty serial no.
normal boot.
checkKey
vbus = 1
board_fbt_key_pressed: ir_keycode = 0x0, frt = 0
no fuel gauge found
no fuel gauge found
Can't find display display route node
read logo on state from dts [0]
no fuel gauge found
checkKey
vbus = 1
board_fbt_key_pressed: ir_keycode = 0x0, frt = 0
Hit any key to stop autoboot:  0 
load fdt from resouce.
vendor read error!
Set oem_unlocked=0Secure Boot state: 0
kernel   @ 0x62000000 (0x0050d378)
ramdisk  @ 0x65bf0000 (0x00156f4e)
bootrk: do_bootm_linux...
   Loading Device Tree to 65600000, end 65608c6c ... OK
Add bank:0000000060000000, 0000000008400000
Add bank:0000000068500000, 0000000037b00000

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0xf00
[    0.000000] Linux version 5.2.0-rc5-dirty (justin@v01.28459.vpscontrol.net) (gcc version 6.3.0 20170516 (Debian 6.3.0-18)) #1 SMP Thu Jun 20 11:52:01 UTC 2019
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Rockchip RK3229 (Mecer Xtreme Mini S6)
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] cma: Reserved 64 MiB at 0x61000000
[    0.000000] On node 0 totalpages: 261888
[    0.000000]   Normal zone: 1536 pages used for memmap
[    0.000000]   Normal zone: 0 pages reserved
[    0.000000]   Normal zone: 196352 pages, LIFO batch:63
[    0.000000]   HighMem zone: 65536 pages, LIFO batch:15
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv65535.65535 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: MIGRATE_INFO_TYPE not supported.
[    0.000000] psci: SMC Calling Convention v1.0
[    0.000000] percpu: Embedded 20 pages/cpu s51788 r8192 d21940 u81920
[    0.000000] pcpu-alloc: s51788 r8192 d21940 u81920 alloc=20*4096
[    0.000000] pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 260352
[    0.000000] Kernel command line: console=ttyS2,1500000n8 console=tty0 debug video=HDMI-A-1:1920x1080@60 mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00004000@0x00004000(trust),0x00002000@0x00008000(misc),0x00000800@0x0000A000(baseparamer),0x00007800@0x0000A800(resource),0x00006000@0x00012000(kernel),0x0000C000@0x00018000(boot),0x00010000@0x00024000(recovery),0x00020000@0x00034000(backup),0x00040000@0x00054000(cache),0x00008000@0x00094000(metadata),0x00002000@0x0009C000(kpanic),0x00400000@0x0009E000(system),-@0x0049E000(userdata) storagemedia=emmc loader.timestamp=2018-07-12_15:15:42 hdmi.vic=-1 tve.format=-1 SecureBootCheckOk=0
[    0.000000] Dentry cache hash table entries: 131072 (order: 7, 524288 bytes)
[    0.000000] Inode-cache hash table entries: 65536 (order: 6, 262144 bytes)
[    0.000000] Memory: 957984K/1047552K available (8192K kernel code, 549K rwdata, 2084K rodata, 1024K init, 267K bss, 24032K reserved, 65536K cma-reserved, 262144K highmem)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=16 to nr_cpu_ids=4.
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
[    0.000022] Switching to timer-based delay loop, resolution 41ns
[    0.000994] Console: colour dummy device 80x30
[    0.001564] printk: console [tty0] enabled
[    0.001624] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=240000)
[    0.001658] pid_max: default: 32768 minimum: 301
[    0.001853] Mount-cache hash table entries: 2048 (order: 1, 8192 bytes)
[    0.001885] Mountpoint-cache hash table entries: 2048 (order: 1, 8192 bytes)
[    0.002655] *** VALIDATE proc ***
[    0.002862] *** VALIDATE cgroup1 ***
[    0.002887] *** VALIDATE cgroup2 ***
[    0.002909] CPU: Testing write buffer coherency: ok
[    0.003443] CPU0: update cpu_capacity 1024
[    0.003480] CPU0: thread -1, cpu 0, socket 15, mpidr 80000f00
[    0.004221] Setting up static identity map for 0x60100000 - 0x60100060
[    0.004419] rcu: Hierarchical SRCU implementation.
[    0.008103] smp: Bringing up secondary CPUs ...
[    0.009127] CPU1: update cpu_capacity 1024
[    0.009136] CPU1: thread -1, cpu 1, socket 15, mpidr 80000f01
[    0.010306] CPU2: update cpu_capacity 1024
[    0.010314] CPU2: thread -1, cpu 2, socket 15, mpidr 80000f02
[    0.011271] CPU3: update cpu_capacity 1024
[    0.011280] CPU3: thread -1, cpu 3, socket 15, mpidr 80000f03
[    0.011414] smp: Brought up 1 node, 4 CPUs
[    0.011519] SMP: Total of 4 processors activated (192.00 BogoMIPS).
[    0.011536] CPU: All CPU(s) started in SVC mode.
[    0.012747] devtmpfs: initialized
[    0.019373] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 5
[    0.019778] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.019836] futex hash table entries: 1024 (order: 4, 65536 bytes)
[    0.022975] pinctrl core: initialized pinctrl subsystem
[    0.024575] NET: Registered protocol family 16
[    0.027393] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.029334] cpuidle: using governor menu
[    0.029886] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.029919] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.030662] Serial: AMBA PL011 UART driver
[    0.063310] vcc_sys: supplied by dc_12v
[    0.063748] vccio_1v8: supplied by vcc_sys
[    0.063999] vcc_host: supplied by vcc_sys
[    0.064267] vccio_3v3: supplied by vcc_sys
[    0.064491] vcc_phy: supplied by vccio_1v8
[    0.066090] SCSI subsystem initialized
[    0.066396] usbcore: registered new interface driver usbfs
[    0.066473] usbcore: registered new interface driver hub
[    0.066616] usbcore: registered new device driver usb
[    0.066918] pps_core: LinuxPPS API ver. 1 registered
[    0.066942] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.066998] EDAC MC: Ver: 3.0.0
[    0.069659] clocksource: Switched to clocksource arch_sys_counter
[    0.130528] NET: Registered protocol family 2
[    0.131402] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 6144 bytes)
[    0.131466] TCP established hash table entries: 8192 (order: 3, 32768 bytes)
[    0.131578] TCP bind hash table entries: 8192 (order: 4, 65536 bytes)
[    0.131730] TCP: Hash tables configured (established 8192 bind 8192)
[    0.131934] UDP hash table entries: 512 (order: 2, 16384 bytes)
[    0.132019] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes)
[    0.132318] NET: Registered protocol family 1
[    0.132391] PCI: CLS 0 bytes, default 64
[    0.132709] Trying to unpack rootfs image as initramfs...
[    0.228227] Freeing initrd memory: 1372K
[    0.229464] hw perfevents: enabled with armv7_cortex_a7 PMU driver, 5 counters available
[    0.231886] Initialise system trusted keyrings
[    0.232287] workingset: timestamp_bits=30 max_order=18 bucket_order=0
[    0.240261] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.240517] ntfs: driver 2.1.32 [Flags: R/O].
[    0.241415] Key type asymmetric registered
[    0.241455] Asymmetric key parser 'x509' registered
[    0.241533] bounce: pool size: 64 pages
[    0.241654] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)
[    0.241684] io scheduler mq-deadline registered
[    0.241700] io scheduler kyber registered
[    0.249604] pwm-regulator: supplied by vcc_sys
[    0.250293] pwm-regulator: supplied by vcc_sys
[    0.316088] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.319186] printk: console [ttyS2] disabled
[    0.319286] 11030000.serial: ttyS2 at MMIO 0x11030000 (irq = 28, base_baud = 1500000) is a 16550A
[    0.383916] printk: console [ttyS2] enabled
[    0.386251] rockchip-vop 20050000.vop: Adding to iommu group 0
[    0.390372] rockchip-drm display-subsystem: bound 20050000.vop (ops 0xc0950168)
[    0.391330] dwhdmi-rockchip 200a0000.hdmi: Detected HDMI TX controller v2.01a with HDCP (inno_dw_hdmi_phy2)
[    0.392817] dwhdmi-rockchip 200a0000.hdmi: registered DesignWare HDMI I2C bus driver
[    0.393894] rockchip-drm display-subsystem: bound 200a0000.hdmi (ops 0xc0953270)
[    0.394575] [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
[    0.395166] [drm] No driver support for vblank timestamp query.
[    0.395759] [drm] Cannot find any crtc or sizes
[    0.396915] [drm] Initialized rockchip 1.0.0 20140818 for display-subsystem on minor 0
[    0.399030] lima 20000000.gpu: bus rate = 148500000
[    0.399500] lima 20000000.gpu: mod rate = 148500000
[    0.401017] lima 20000000.gpu: gp - mali400 version major 1 minor 1
[    0.401691] lima 20000000.gpu: pp0 - mali400 version major 1 minor 1
[    0.402324] lima 20000000.gpu: pp1 - mali400 version major 1 minor 1
[    0.402919] lima 20000000.gpu: l2 cache 64K, 4-way, 64byte cache line, 64bit external bus
[    0.404775] [drm] Initialized lima 1.0.0 20190217 for 20000000.gpu on minor 1
[    0.418713] brd: module loaded
[    0.430945] loop: module loaded
[    0.432709] libphy: Fixed MDIO Bus: probed
[    0.433911] rk_gmac-dwmac 30200000.ethernet: PTP uses main clock
[    0.434590] rk_gmac-dwmac 30200000.ethernet: clock input or output? (output).
[    0.435236] rk_gmac-dwmac 30200000.ethernet: Can not read property: tx_delay.
[    0.435877] rk_gmac-dwmac 30200000.ethernet: set tx_delay to 0x30
[    0.436426] rk_gmac-dwmac 30200000.ethernet: Can not read property: rx_delay.
[    0.437065] rk_gmac-dwmac 30200000.ethernet: set rx_delay to 0x10
[    0.437622] rk_gmac-dwmac 30200000.ethernet: integrated PHY? (yes).
[    0.438253] rk_gmac-dwmac 30200000.ethernet: cannot get clock clk_mac_speed
[    0.443917] rk_gmac-dwmac 30200000.ethernet: init for RMII
[    0.489960] rk_gmac-dwmac 30200000.ethernet: User ID: 0x10, Synopsys ID: 0x35
[    0.490640] rk_gmac-dwmac 30200000.ethernet:         DWMAC1000
[    0.491119] rk_gmac-dwmac 30200000.ethernet: DMA HW capability register supported
[    0.491794] rk_gmac-dwmac 30200000.ethernet: RX Checksum Offload Engine supported
[    0.492466] rk_gmac-dwmac 30200000.ethernet: COE Type 2
[    0.492938] rk_gmac-dwmac 30200000.ethernet: TX Checksum insertion supported
[    0.493567] rk_gmac-dwmac 30200000.ethernet: Wake-Up On Lan supported
[    0.494203] rk_gmac-dwmac 30200000.ethernet: Normal descriptors
[    0.494738] rk_gmac-dwmac 30200000.ethernet: Ring mode enabled
[    0.495263] rk_gmac-dwmac 30200000.ethernet: Enable RX Mitigation via HW Watchdog Timer
[    0.496002] rk_gmac-dwmac 30200000.ethernet (unnamed net_device) (uninitialized): device MAC address 3a:9c:1a:23:72:c5
[    0.497164] libphy: stmmac: probed
[    0.504299] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.504941] ehci-pci: EHCI PCI platform driver
[    0.505421] ehci-platform: EHCI generic platform driver
[    0.508345] ehci-platform 30080000.usb: EHCI Host Controller
[    0.508914] ehci-platform 30080000.usb: new USB bus registered, assigned bus number 1
[    0.510088] ehci-platform 30080000.usb: irq 39, io mem 0x30080000
[    0.539683] ehci-platform 30080000.usb: USB 2.0 started, EHCI 1.00
[    0.541441] hub 1-0:1.0: USB hub found
[    0.541862] hub 1-0:1.0: 1 port detected
[    0.545115] ehci-platform 300c0000.usb: EHCI Host Controller
[    0.545680] ehci-platform 300c0000.usb: new USB bus registered, assigned bus number 2
[    0.546832] ehci-platform 300c0000.usb: irq 41, io mem 0x300c0000
[    0.579663] ehci-platform 300c0000.usb: USB 2.0 started, EHCI 1.00
[    0.581391] hub 2-0:1.0: USB hub found
[    0.581816] hub 2-0:1.0: 1 port detected
[    0.585040] ehci-platform 30100000.usb: EHCI Host Controller
[    0.585603] ehci-platform 30100000.usb: new USB bus registered, assigned bus number 3
[    0.586735] ehci-platform 30100000.usb: irq 43, io mem 0x30100000
[    0.619665] ehci-platform 30100000.usb: USB 2.0 started, EHCI 1.00
[    0.621317] hub 3-0:1.0: USB hub found
[    0.621750] hub 3-0:1.0: 1 port detected
[    0.622775] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.623370] ohci-platform: OHCI generic platform driver
[    0.624240] ohci-platform 300a0000.usb: Generic Platform OHCI controller
[    0.624883] ohci-platform 300a0000.usb: new USB bus registered, assigned bus number 4
[    0.625916] ohci-platform 300a0000.usb: irq 40, io mem 0x300a0000
[    0.695482] hub 4-0:1.0: USB hub found
[    0.695906] hub 4-0:1.0: 1 port detected
[    0.697071] ohci-platform 300e0000.usb: Generic Platform OHCI controller
[    0.697717] ohci-platform 300e0000.usb: new USB bus registered, assigned bus number 5
[    0.698772] ohci-platform 300e0000.usb: irq 42, io mem 0x300e0000
[    0.764723] hub 5-0:1.0: USB hub found
[    0.765145] hub 5-0:1.0: 1 port detected
[    0.766313] ohci-platform 30120000.usb: Generic Platform OHCI controller
[    0.766963] ohci-platform 30120000.usb: new USB bus registered, assigned bus number 6
[    0.768038] ohci-platform 30120000.usb: irq 44, io mem 0x30120000
[    0.834688] hub 6-0:1.0: USB hub found
[    0.835116] hub 6-0:1.0: 1 port detected
[    0.836733] usbcore: registered new interface driver usb-storage
[    0.837674] i2c /dev entries driver
[    0.840722] rockchip-thermal 11150000.tsadc: Missing tshut-polarity property, using default (low)
[    0.841553] rockchip-thermal 11150000.tsadc: Missing rockchip,grf property
[    0.843391] softdog: initialized. soft_noboot=0 soft_margin=60 sec soft_panic=0 (nowayout=0)
[    0.844506] Synopsys Designware Multimedia Card Interface Driver
[    0.845785] dwmmc_rockchip 30000000.dwmmc: IDMAC supports 32-bit address mode.
[    0.846649] dwmmc_rockchip 30000000.dwmmc: Using internal DMA controller.
[    0.847275] dwmmc_rockchip 30000000.dwmmc: Version ID is 270a
[    0.847868] dwmmc_rockchip 30000000.dwmmc: DW MMC controller at irq 36,32 bit host data width,256 deep fifo
[    0.859802] mmc_host mmc0: Bus speed (slot 0) = 400000Hz (slot req 400000Hz, actual 400000HZ div = 0)
[    0.874126] dwmmc_rockchip 30020000.dwmmc: IDMAC supports 32-bit address mode.
[    0.875028] dwmmc_rockchip 30020000.dwmmc: Using internal DMA controller.
[    0.875682] dwmmc_rockchip 30020000.dwmmc: Version ID is 270a
[    0.876303] dwmmc_rockchip 30020000.dwmmc: DW MMC controller at irq 37,32 bit host data width,256 deep fifo
[    0.877343] mmc_host mmc1: card is non-removable.
[    0.890591] mmc_host mmc1: Bus speed (slot 0) = 1160156Hz (slot req 400000Hz, actual 290039HZ div = 2)
[    0.905169] ledtrig-cpu: registered to indicate activity on CPUs
[    0.906010] usbcore: registered new interface driver usbhid
[    0.906562] usbhid: USB HID core driver
[    0.907401] ashmem: initialized
[    0.910937] NET: Registered protocol family 10
[    0.912728] Segment Routing with IPv6
[    0.913237] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    0.914696] NET: Registered protocol family 17
[    0.915327] Key type dns_resolver registered
[    0.916035] ThumbEE CPU extension supported.
[    0.916438] Registering SWP/SWPB emulation handler
[    0.917817] Loading compiled-in X.509 certificates
[    0.934518] hctosys: unable to open rtc device (rtc0)
[    0.937196] Freeing unused kernel memory: 1024K
[    0.989919] Run /init as init process
[    1.000828] mmc_host mmc1: Bus speed (slot 0) = 37125000Hz (slot req 37500000Hz, actual 37125000HZ div = 0)
[    1.003108] mmc1: new high speed MMC card at address 0001
[    1.006375] mmcblk1: mmc1:0001 NCard  7.28 GiB 
[    1.008832] mmcblk1boot0: mmc1:0001 NCard  partition 1 4.00 MiB
[    1.011516] mmcblk1boot1: mmc1:0001 NCard  partition 2 4.00 MiB
[    1.012587] mmcblk1rpmb: mmc1:0001 NCard  partition 3 4.00 MiB, chardev (246:0)
[    1.019367] ttyS2 - failed to request DMA
[    1.449800] [drm] Cannot find any crtc or sizes



BusyBox v1.22.1 (Debian 1:1.22.0-19+b3) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/ #
```


### Create a GPT partition table

```
/ # fdisk -b 512 /dev/mmcblk1

Welcome to fdisk (util-linux 2.29.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.

Created a new DOS disklabel with disk identifier 0x9cd07133.

Command (m for help): g
Created a new GPT disklabel (GUID: 68247CC7-EC1C-4933-BF43-C46D090DF20F).

Command (m for help): p
Disk /dev/mmcblk1: 7.3 GiB, 7818182656 bytes, 15269888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: FA0B3C71-05D6-4787-B89B-8D871F0A9D32

```


### Add the root filesystem partition

The kernel needs to detect a partition on the eMMC that corresponds to where the root filesystem will be stored. Android uses the **system** partition declared in mtdparts for root filesystem storage, so it is a good idea to create a partition that will cover the same region of the disk. If the reminder of the disk is allocated for Android userdata, it can be combined into the allocation too.

Note that the partition offsets in the mtdparts declarations are listed as logical block addresses and are 0x2000 sectors less than the actual location on disk! For example, **0x00400000@0x0009E000(system)** represents the system partition at sector 0x9e000 + 0x2000, or 0xa0000.

Unfortunately, you will need to manually convert all sector offsets to decimal for input into fdisk.

```
Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-15269854, default 2048): 655360
Last sector, +sectors or +size{K,M,G,T,P} (655360-15269854, default 15269854): 

Created a new partition 1 of type 'Linux filesystem' and of size 7 GiB.
Partition #1 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: y

The signature will be removed by a write command.

Command (m for help): p
Disk /dev/mmcblk1: 7.3 GiB, 7818182656 bytes, 15269888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C9F230AF-B578-4024-ADC8-8BF1C05F3DBE

Device          Start      End  Sectors Size Type
/dev/mmcblk1p1 655360 15269854 14614495   7G Linux filesystem

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```


### Generate the system image

```
# dd if=/dev/zero of=$REPLACEMENT/system.img bs=512 of=14614495
# mke2fs -t ext2 system.img
# mount -o loop system.img $SYSTEM
# cp -ar $ROOTFS/* $SYSTEM/
```

### Update the kernel command line parameters

Be sure to adjust the value of the mtdparts variable to reflect the new size of **system** partition if it has increased in size, and then remove any subsequent partition declarations that have been overlapped.

```
# grep '^CMDLINE' $REPLACEMENT/parameters
CMDLINE:console=ttyS2,1500000n8 console=tty0 debug video=HDMI-A-1:1920x1080@60 mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00004000@0x00004000(trust),0x00002000@0x00008000(misc),0x00000800@0x0000A000(baseparamer),0x00007800@0x0000A800(resource),0x00006000@0x00012000(kernel),0x0000C000@0x00018000(boot),0x00010000@0x00024000(recovery),0x00020000@0x00034000(backup),0x00040000@0x00054000(cache),0x00008000@0x00094000(metadata),0x00002000@0x0009C000(kpanic),-@0x0009E000(system)
```


### Push the system image and updated parameters

```
# rkflashtool w system < $REPLACEMENT/system.img
# rkflashtool P < $REPLACEMENT/parameters
```


### Reboot and access the HDMI or UART2 console

```
Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0xf00
[    0.000000] Linux version 5.2.0-rc5-dirty (justin@v01.28459.vpscontrol.net) (gcc version 6.3.0 20170516 (Debian 6.3.0-18)) #1 SMP Thu Jun 20 11:52:01 UTC 2019
[    0.000000] CPU: ARMv7 Processor [410fc075] revision 5 (ARMv7), cr=10c5387d
[    0.000000] CPU: div instructions available: patching division code
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Rockchip RK3229 (Mecer Xtreme Mini S6)
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] cma: Reserved 64 MiB at 0x61000000
[    0.000000] On node 0 totalpages: 261888
[    0.000000]   Normal zone: 1536 pages used for memmap
[    0.000000]   Normal zone: 0 pages reserved
[    0.000000]   Normal zone: 196352 pages, LIFO batch:63
[    0.000000]   HighMem zone: 65536 pages, LIFO batch:15
[    0.000000] psci: probing for conduit method from DT.
[    0.000000] psci: PSCIv65535.65535 detected in firmware.
[    0.000000] psci: Using standard PSCI v0.2 function IDs
[    0.000000] psci: MIGRATE_INFO_TYPE not supported.
[    0.000000] psci: SMC Calling Convention v1.0
[    0.000000] percpu: Embedded 20 pages/cpu s51788 r8192 d21940 u81920
[    0.000000] pcpu-alloc: s51788 r8192 d21940 u81920 alloc=20*4096
[    0.000000] pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 260352
[    0.000000] Kernel command line: console=ttyS2,1500000n8 console=tty0 debug video=HDMI-A-1:1920x1080@60 mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00004000@0x00004000(trust),0x00002000@0x00008000(misc),0x00000800@0x0000A000(baseparamer),0x00007800@0x0000A800(resource),0x00006000@0x00012000(kernel),0x0000C000@0x00018000(boot),0x00010000@0x00024000(recovery),0x00020000@0x00034000(backup),0x00040000@0x00054000(cache),0x00008000@0x00094000(metadata),0x00002000@0x0009C000(kpanic),-@0x0009E000(system) storagemedia=emmc loader.timestamp=2018-07-12_15:15:42 hdmi.vic=-1 tve.format=-1 SecureBootCheckOk=0
[    0.000000] Dentry cache hash table entries: 131072 (order: 7, 524288 bytes)
[    0.000000] Inode-cache hash table entries: 65536 (order: 6, 262144 bytes)
[    0.000000] Memory: 957728K/1047552K available (8192K kernel code, 549K rwdata, 2084K rodata, 1024K init, 267K bss, 24288K reserved, 65536K cma-reserved, 262144K highmem)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] rcu: Hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=16 to nr_cpu_ids=4.
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
[    0.000990] Console: colour dummy device 80x30
[    0.001549] printk: console [tty0] enabled
[    0.001611] Calibrating delay loop (skipped), value calculated using timer frequency.. 48.00 BogoMIPS (lpj=240000)
[    0.001644] pid_max: default: 32768 minimum: 301
[    0.001838] Mount-cache hash table entries: 2048 (order: 1, 8192 bytes)
[    0.001870] Mountpoint-cache hash table entries: 2048 (order: 1, 8192 bytes)
[    0.002641] *** VALIDATE proc ***
[    0.002849] *** VALIDATE cgroup1 ***
[    0.002875] *** VALIDATE cgroup2 ***
[    0.002899] CPU: Testing write buffer coherency: ok
[    0.003428] CPU0: update cpu_capacity 1024
[    0.003460] CPU0: thread -1, cpu 0, socket 15, mpidr 80000f00
[    0.004178] Setting up static identity map for 0x60100000 - 0x60100060
[    0.004378] rcu: Hierarchical SRCU implementation.
[    0.008065] smp: Bringing up secondary CPUs ...
[    0.009089] CPU1: update cpu_capacity 1024
[    0.009100] CPU1: thread -1, cpu 1, socket 15, mpidr 80000f01
[    0.010262] CPU2: update cpu_capacity 1024
[    0.010272] CPU2: thread -1, cpu 2, socket 15, mpidr 80000f02
[    0.011243] CPU3: update cpu_capacity 1024
[    0.011252] CPU3: thread -1, cpu 3, socket 15, mpidr 80000f03
[    0.011388] smp: Brought up 1 node, 4 CPUs
[    0.011491] SMP: Total of 4 processors activated (192.00 BogoMIPS).
[    0.011508] CPU: All CPU(s) started in SVC mode.
[    0.012728] devtmpfs: initialized
[    0.019299] VFP support v0.3: implementor 41 architecture 2 part 30 variant 7 rev 5
[    0.019706] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.019765] futex hash table entries: 1024 (order: 4, 65536 bytes)
[    0.022919] pinctrl core: initialized pinctrl subsystem
[    0.024523] NET: Registered protocol family 16
[    0.027334] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.029293] cpuidle: using governor menu
[    0.029831] hw-breakpoint: found 5 (+1 reserved) breakpoint and 4 watchpoint registers.
[    0.029864] hw-breakpoint: maximum watchpoint size is 8 bytes.
[    0.030620] Serial: AMBA PL011 UART driver
[    0.063322] vcc_sys: supplied by dc_12v
[    0.063765] vccio_1v8: supplied by vcc_sys
[    0.064012] vcc_host: supplied by vcc_sys
[    0.064284] vccio_3v3: supplied by vcc_sys
[    0.064509] vcc_phy: supplied by vccio_1v8
[    0.066093] SCSI subsystem initialized
[    0.066394] usbcore: registered new interface driver usbfs
[    0.066473] usbcore: registered new interface driver hub
[    0.066617] usbcore: registered new device driver usb
[    0.066911] pps_core: LinuxPPS API ver. 1 registered
[    0.066933] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.066989] EDAC MC: Ver: 3.0.0
[    0.069624] clocksource: Switched to clocksource arch_sys_counter
[    0.130398] NET: Registered protocol family 2
[    0.131268] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 6144 bytes)
[    0.131336] TCP established hash table entries: 8192 (order: 3, 32768 bytes)
[    0.131445] TCP bind hash table entries: 8192 (order: 4, 65536 bytes)
[    0.131597] TCP: Hash tables configured (established 8192 bind 8192)
[    0.131803] UDP hash table entries: 512 (order: 2, 16384 bytes)
[    0.131891] UDP-Lite hash table entries: 512 (order: 2, 16384 bytes)
[    0.132195] NET: Registered protocol family 1
[    0.132270] PCI: CLS 0 bytes, default 64
[    0.132586] Trying to unpack rootfs image as initramfs...
[    0.245884] Freeing initrd memory: 1628K
[    0.247066] hw perfevents: enabled with armv7_cortex_a7 PMU driver, 5 counters available
[    0.249412] Initialise system trusted keyrings
[    0.249921] workingset: timestamp_bits=30 max_order=18 bucket_order=0
[    0.257617] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.257872] ntfs: driver 2.1.32 [Flags: R/O].
[    0.258663] Key type asymmetric registered
[    0.258698] Asymmetric key parser 'x509' registered
[    0.258785] bounce: pool size: 64 pages
[    0.258866] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 247)
[    0.258893] io scheduler mq-deadline registered
[    0.258909] io scheduler kyber registered
[    0.266798] pwm-regulator: supplied by vcc_sys
[    0.267355] pwm-regulator: supplied by vcc_sys
[    0.335654] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.338895] printk: console [ttyS2] disabled
[    0.338996] 11030000.serial: ttyS2 at MMIO 0x11030000 (irq = 28, base_baud = 1500000) is a 16550A
[    0.403292] printk: console [ttyS2] enabled
[    0.405566] rockchip-vop 20050000.vop: Adding to iommu group 0
[    0.409506] rockchip-drm display-subsystem: bound 20050000.vop (ops 0xc0950168)
[    0.410560] dwhdmi-rockchip 200a0000.hdmi: Detected HDMI TX controller v2.01a with HDCP (inno_dw_hdmi_phy2)
[    0.412026] dwhdmi-rockchip 200a0000.hdmi: registered DesignWare HDMI I2C bus driver
[    0.413119] rockchip-drm display-subsystem: bound 200a0000.hdmi (ops 0xc0953270)
[    0.413798] [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
[    0.414389] [drm] No driver support for vblank timestamp query.
[    0.414986] [drm] Cannot find any crtc or sizes
[    0.416136] [drm] Initialized rockchip 1.0.0 20140818 for display-subsystem on minor 0
[    0.418203] lima 20000000.gpu: bus rate = 148500000
[    0.418677] lima 20000000.gpu: mod rate = 148500000
[    0.420176] lima 20000000.gpu: gp - mali400 version major 1 minor 1
[    0.420823] lima 20000000.gpu: pp0 - mali400 version major 1 minor 1
[    0.421462] lima 20000000.gpu: pp1 - mali400 version major 1 minor 1
[    0.422057] lima 20000000.gpu: l2 cache 64K, 4-way, 64byte cache line, 64bit external bus
[    0.423946] [drm] Initialized lima 1.0.0 20190217 for 20000000.gpu on minor 1
[    0.437673] brd: module loaded
[    0.449866] loop: module loaded
[    0.451660] libphy: Fixed MDIO Bus: probed
[    0.452895] rk_gmac-dwmac 30200000.ethernet: PTP uses main clock
[    0.453581] rk_gmac-dwmac 30200000.ethernet: clock input or output? (output).
[    0.454230] rk_gmac-dwmac 30200000.ethernet: Can not read property: tx_delay.
[    0.454870] rk_gmac-dwmac 30200000.ethernet: set tx_delay to 0x30
[    0.455419] rk_gmac-dwmac 30200000.ethernet: Can not read property: rx_delay.
[    0.456057] rk_gmac-dwmac 30200000.ethernet: set rx_delay to 0x10
[    0.456616] rk_gmac-dwmac 30200000.ethernet: integrated PHY? (yes).
[    0.457249] rk_gmac-dwmac 30200000.ethernet: cannot get clock clk_mac_speed
[    0.462916] rk_gmac-dwmac 30200000.ethernet: init for RMII
[    0.509927] rk_gmac-dwmac 30200000.ethernet: User ID: 0x10, Synopsys ID: 0x35
[    0.510594] rk_gmac-dwmac 30200000.ethernet:         DWMAC1000
[    0.511070] rk_gmac-dwmac 30200000.ethernet: DMA HW capability register supported
[    0.511741] rk_gmac-dwmac 30200000.ethernet: RX Checksum Offload Engine supported
[    0.512413] rk_gmac-dwmac 30200000.ethernet: COE Type 2
[    0.512885] rk_gmac-dwmac 30200000.ethernet: TX Checksum insertion supported
[    0.513515] rk_gmac-dwmac 30200000.ethernet: Wake-Up On Lan supported
[    0.514144] rk_gmac-dwmac 30200000.ethernet: Normal descriptors
[    0.514677] rk_gmac-dwmac 30200000.ethernet: Ring mode enabled
[    0.515203] rk_gmac-dwmac 30200000.ethernet: Enable RX Mitigation via HW Watchdog Timer
[    0.515939] rk_gmac-dwmac 30200000.ethernet (unnamed net_device) (uninitialized): device MAC address 0e:0e:47:19:84:fb
[    0.517108] libphy: stmmac: probed
[    0.524262] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    0.524901] ehci-pci: EHCI PCI platform driver
[    0.525377] ehci-platform: EHCI generic platform driver
[    0.528297] ehci-platform 30080000.usb: EHCI Host Controller
[    0.528870] ehci-platform 30080000.usb: new USB bus registered, assigned bus number 1
[    0.530102] ehci-platform 30080000.usb: irq 39, io mem 0x30080000
[    0.559629] ehci-platform 30080000.usb: USB 2.0 started, EHCI 1.00
[    0.561394] hub 1-0:1.0: USB hub found
[    0.561817] hub 1-0:1.0: 1 port detected
[    0.565059] ehci-platform 300c0000.usb: EHCI Host Controller
[    0.565621] ehci-platform 300c0000.usb: new USB bus registered, assigned bus number 2
[    0.566672] ehci-platform 300c0000.usb: irq 41, io mem 0x300c0000
[    0.589624] ehci-platform 300c0000.usb: USB 2.0 started, EHCI 1.00
[    0.591277] hub 2-0:1.0: USB hub found
[    0.591700] hub 2-0:1.0: 1 port detected
[    0.594928] ehci-platform 30100000.usb: EHCI Host Controller
[    0.595492] ehci-platform 30100000.usb: new USB bus registered, assigned bus number 3
[    0.596577] ehci-platform 30100000.usb: irq 43, io mem 0x30100000
[    0.619627] ehci-platform 30100000.usb: USB 2.0 started, EHCI 1.00
[    0.621267] hub 3-0:1.0: USB hub found
[    0.621688] hub 3-0:1.0: 1 port detected
[    0.622677] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    0.623290] ohci-platform: OHCI generic platform driver
[    0.624168] ohci-platform 300a0000.usb: Generic Platform OHCI controller
[    0.624815] ohci-platform 300a0000.usb: new USB bus registered, assigned bus number 4
[    0.625899] ohci-platform 300a0000.usb: irq 40, io mem 0x300a0000
[    0.694704] hub 4-0:1.0: USB hub found
[    0.695133] hub 4-0:1.0: 1 port detected
[    0.696301] ohci-platform 300e0000.usb: Generic Platform OHCI controller
[    0.696945] ohci-platform 300e0000.usb: new USB bus registered, assigned bus number 5
[    0.697968] ohci-platform 300e0000.usb: irq 42, io mem 0x300e0000
[    0.764718] hub 5-0:1.0: USB hub found
[    0.765145] hub 5-0:1.0: 1 port detected
[    0.766320] ohci-platform 30120000.usb: Generic Platform OHCI controller
[    0.766966] ohci-platform 30120000.usb: new USB bus registered, assigned bus number 6
[    0.768021] ohci-platform 30120000.usb: irq 44, io mem 0x30120000
[    0.834656] hub 6-0:1.0: USB hub found
[    0.835083] hub 6-0:1.0: 1 port detected
[    0.836690] usbcore: registered new interface driver usb-storage
[    0.837644] i2c /dev entries driver
[    0.840665] rockchip-thermal 11150000.tsadc: Missing tshut-polarity property, using default (low)
[    0.841494] rockchip-thermal 11150000.tsadc: Missing rockchip,grf property
[    0.843327] softdog: initialized. soft_noboot=0 soft_margin=60 sec soft_panic=0 (nowayout=0)
[    0.844467] Synopsys Designware Multimedia Card Interface Driver
[    0.845733] dwmmc_rockchip 30000000.dwmmc: IDMAC supports 32-bit address mode.
[    0.846577] dwmmc_rockchip 30000000.dwmmc: Using internal DMA controller.
[    0.847202] dwmmc_rockchip 30000000.dwmmc: Version ID is 270a
[    0.847793] dwmmc_rockchip 30000000.dwmmc: DW MMC controller at irq 36,32 bit host data width,256 deep fifo
[    0.861709] mmc_host mmc0: Bus speed (slot 0) = 400000Hz (slot req 400000Hz, actual 400000HZ div = 0)
[    0.876026] dwmmc_rockchip 30020000.dwmmc: IDMAC supports 32-bit address mode.
[    0.876943] dwmmc_rockchip 30020000.dwmmc: Using internal DMA controller.
[    0.877599] dwmmc_rockchip 30020000.dwmmc: Version ID is 270a
[    0.878194] dwmmc_rockchip 30020000.dwmmc: DW MMC controller at irq 37,32 bit host data width,256 deep fifo
[    0.879215] mmc_host mmc1: card is non-removable.
[    0.892565] mmc_host mmc1: Bus speed (slot 0) = 1160156Hz (slot req 400000Hz, actual 290039HZ div = 2)
[    0.907211] ledtrig-cpu: registered to indicate activity on CPUs
[    0.908052] usbcore: registered new interface driver usbhid
[    0.908613] usbhid: USB HID core driver
[    0.909433] ashmem: initialized
[    0.912990] NET: Registered protocol family 10
[    0.914884] Segment Routing with IPv6
[    0.915332] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    0.916743] NET: Registered protocol family 17
[    0.917374] Key type dns_resolver registered
[    0.918029] ThumbEE CPU extension supported.
[    0.918430] Registering SWP/SWPB emulation handler
[    0.919993] Loading compiled-in X.509 certificates
[    0.936537] hctosys: unable to open rtc device (rtc0)
[    0.939329] Freeing unused kernel memory: 1024K
[    0.979870] Run /init as init process
[    1.023179] mmc_host mmc1: Bus speed (slot 0) = 37125000Hz (slot req 37500000Hz, actual 37125000HZ div = 0)
[    1.025164] mmc1: new high speed MMC card at address 0001
[    1.027839] mmcblk1: mmc1:0001 NCard  7.28 GiB 
[    1.030025] mmcblk1boot0: mmc1:0001 NCard  partition 1 4.00 MiB
[    1.032145] mmcblk1boot1: mmc1:0001 NCard  partition 2 4.00 MiB
[    1.033172] mmcblk1rpmb: mmc1:0001 NCard  partition 3 4.00 MiB, chardev (246:0)
[    1.042084]  mmcblk1: p1
[    1.449753] [drm] Cannot find any crtc or sizes
[    2.008958] EXT4-fs (mmcblk1p1): mounting ext2 file system using the ext4 subsystem
[    2.016449] EXT4-fs (mmcblk1p1): warning: mounting unchecked fs, running e2fsck is recommended
[    2.019235] EXT4-fs (mmcblk1p1): mounted filesystem without journal. Opts: (null)
[    2.089568] random: fast init done
[    2.818269] udevd[202]: starting version 3.2.2
[    2.838548] random: udevd: uninitialized urandom read (16 bytes read)
[    2.841138] random: udevd: uninitialized urandom read (16 bytes read)
[    2.841860] random: udevd: uninitialized urandom read (16 bytes read)
[    2.895469] udevd[203]: starting eudev-3.2.2
[    4.125254] EXT4-fs (mmcblk1p1): re-mounted. Opts: (null)
[    5.594209] urandom_read: 2 callbacks suppressed
[    5.594222] random: dd: uninitialized urandom read (512 bytes read)
[    6.449334] ttyS2 - failed to request DMA

workstation login: 
```
