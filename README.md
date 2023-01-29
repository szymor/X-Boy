# Open source game handheld based on Allwinner V3s

Original text by hsinyuwang  
Translation and editing by vamastah

Video presentation (Chinese) [https://www.bilibili.com/video/BV1JP4y1X7Uz](https://www.bilibili.com/video/BV1JP4y1X7Uz/?vd_source=2f25de86752ccc9cfafc4bcf1c352db1)

Pictures

![](/images/7.jpg)

![](/images/6.jpg)

Below you can find instructions on how to build a binary image of firmware to be written to an SD card. You can also use the image created by hsinyuwang:

> Baidu storage
> 
> Link: https://pan.baidu.com/s/1ftacGCXHLy-AFabczBg3yg
>
> Extraction code: 9bp8

I could not manage to download the image from the link above, so I (vamastah) created another one from scratch and published in Releases section of this repository. It was not tested yet, so if you happen to have any issues, please inform me.

**Guide on firmware image construction for X-Boy**

- [Ubuntu installation](#ubuntu-installation)
	- [Download mirror](#download-mirror)
	- [Change APT package source](#change-apt-package-source)
		- [backup the source list](#backup-the-source-list)
		- [modify sources.list](#modify-sourceslist)
		- [change to Tsinghua mirror source](#change-to-tsinghua-mirror-source)
		- [update and upgrade packages](#update-and-upgrade-packages)
		- [install dependencies](#install-dependencies)
- [Installation of the cross-compilation toolchain](#installation-of-the-cross-compilation-toolchain)
	- [Install the compiler](#install-the-compiler)
		- [create a folder](#create-a-folder)
		- [download the precompiled toolchain](#download-the-precompiled-toolchain)
		- [untar the toolchain](#untar-the-toolchain)
		- [configure environment variables](#configure-environment-variables)
		- [install toolchain dependencies](#install-toolchain-dependencies)
		- [verify the installation](#verify-the-installation)
- [U-Boot compilation](#u-boot-compilation)
	- [download U-Boot](#download-u-boot)
	- [modify include/configs/sun8i.h](#modify-includeconfigssun8ih)
	- [compile U-Boot](#compile-uboot)
- [Linux compilation](#linux-compilation)
	- [download source code](#download-source-code)
	- [modify the top-level Makefile](#modify-the-top-level-makefile)
	- [configure ILI9341 LCD](#configure-ili9341-lcd)
	- [compile](#compile)
- [Buildroot rootfs building](#buildroot-根文件系统构建)
	- [get Buildroot](#获取-buildroot)
	- [basic configuration](#基本配置)
	- [toolchain configuration](#编译链工具配置)
	- [alsa, sdl, fbv configuration](#alsasdlfbv配置)
	- [build](#编译-1)
- [SD card partitioning](#tf-卡分区及烧录)
	- [partition the SD card](#tf-卡分区)
	- [burn U-Boot](#烧录-uboot)
	- [copy the kernel and the device tree](#写入内核和设备树)
	- [untar rootfs](#写入根文件系统)
- [Emulator compilation](#编译模拟器)
	- [compile gpSP](#编译-gpsp)
	- [copy the executable](#复制可执行文件)
- [System configuration](#系统配置)
	- [automount fat partition](#自动挂载-fat-分区)
	- [configure terminal display](#配置双端显示)
	- [turn on the sound after startup](#启动后开启声音)
	- [configure SDL environment](#配置-sdl-环境)
	- [autostart the emulator](#自启动模拟器)
- [Image mirroring](#制作镜像)
	- [create a working directory](#创建工作目录)
	- [create a blank file and partition it](#创建空白文件并分区)
	- [map the image into block devices](#将镜像文件虚拟成块设备)
	- [format the block devices and mount](#格式化块设备并且挂载)
	- [burn U-Boot](#烧录-uboot-1)
	- [copy the kernel and the device tree](#写入内核和设备树-1)
	- [untar rootfs](#写入根文件系统-1)
- [Acknowledgements](#acknowledgements)
- [References](#references)

---

## Ubuntu installation

### Download mirror

```
https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/18.04/ubuntu-18.04.6-desktop-amd64.iso
```

You can install it on a physical machine or on a VM. I suggest using VirtualBox as it offers an unattended installation feature - really convenient!

### Change APT package source

#### backup the source list

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```
#### modify sources.list

```
sudo gedit /etc/apt/sources.list
```

#### change to Tsinghua mirror source

```
# repositories with sources are commented out to improve the speed of apt update, uncomment if necessary
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# pre-release software repository, not recommended
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

#### update and upgrade packages

```
sudo apt-get update && sudo apt-get upgrade
```

#### install dependencies

```
sudo apt-get install -y device-tree-compiler python flex bison ncurses-dev libssl-dev
```

## Installation of the cross-compilation toolchain

### Install the compiler

#### create a folder

```
mkdir -p ~/linux/tools && cd ~/linux/tools
```

#### download the precompiled toolchain

```
wget https://releases.linaro.org/components/toolchain/binaries/4.9-2017.01/arm-linux-gnueabihf/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz
```

#### untar the toolchain

```
sudo mkdir /usr/local/arm && sudo tar -vxf gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf.tar.xz -C /usr/local/arm
```

#### configure environment variables

```
# run a text editor as root
sudo vim ~/.bashrc

# write the line below in the file
export PATH=$PATH:/usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin

# execute in the terminal
source ~/.bashrc
```
#### install toolchain dependencies

```
sudo apt-get install lsb-core lib32stdc++6
```

#### verify the installation

```
arm-linux-gnueabihf-gcc -v
```

![](/images/1.jpg)

## U-boot compilation

### download uboot

```
mkdir ~/v3s && cd ~/v3s
git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current
```
Directory structure of U-Boot

```
├── api                the interface of the functions provided by U-Boot
├── arch               platform-related parts, we only care about the 'arm' subfolder
│   ├──arm
│   │   └──cpu
│   │   │   └──armv7
│   │   └──dts
│   │   │   └──*.dts   device tree source, e.g. pin information of the device
├── board              source code specific for development boards
├── cmd                implementation of U-Boot commands
├── common             common code
├── configs            configuration files for different devboards (including Lichee)
├── disk               implementation of some disk operations such as partitioning
├── doc                reference documentation, including platform-related documents
├── drivers            source code of drivers
├── dts                Kconfig and build scripts for device tree
├── examples           official sample programs
├── fs                 implementation of different file systems supported by U-Boot
├── include            header files
├── lib                commonly used libraries
├── Licenses           licenses, legal documents
├── net                network stack
├── post               power-on self-test program
├── scripts            compile scripts and makefiles
├── spl                secondary program loader，the second stage of bootloader
├── test               unit tests
└── tools              tools commonly used by U-Boot
```
### modify include/configs/sun8i.h

In the file add:

```
#define CONFIG_BOOTCOMMAND  "setenv bootm_boot_mode sec; " \
                            "load mmc 0:1 0x41000000 zImage; "  \
                            "load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero-dock.dtb; " \
                            "bootz 0x41000000 - 0x41800000;"

#define CONFIG_BOOTARGS "console=tty0 console=ttyS0,115200 panic=5 rootwait root=/dev/mmcblk0p2 earlyprintk rw  vt.global_cursor_default=0"
```

![](/images/2.jpg)

### compile U-Boot

```
cd u-boot
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_defconfig
make ARCH=arm menuconfig
time make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- 2>&1 | tee build.log
```

If you add the default compiler in line 248 of the top-level Makefile, then you will be able to compile U-Boot by calling 'make' with no additional parameters.

```
# set default to nothing for native builds
ifeq ($(HOSTARCH),$(ARCH))
CROSS_COMPILE ?= 
endif

ARCH  ?= arm
CROSS_COMPILE ?= arm-linux-gnueabihf-

KCONFIG_CONFIG	?= .config
export KCONFIG_CONFIG
```

Compile successfully.

![](/images/3.jpg)

The kernel and rootfs will be built in the later steps.

## Linux compilation

### download source code

```
cd ~/v3s
git clone -b zero-5.2.y https://github.com/Lichee-Pi/linux.git
```

### modify the top-level Makefile

If you add the default compiler in line 364 of the top-level Makefile, you will be able to compile Linux by calling 'make' with no additional parameters.

```
# ARCH		?= $(SUBARCH)
ARCH		?= arm
CROSS_COMPILE	?= arm-linux-gnueabihf-
```

### configure ILI9341 LCD

```
cd linux
make licheepi_zero_defconfig
make menuconfig
```

Select the following drivers:

```
Device Drivers  --->
    Input device support  --->
        <*>   Joystick interface
        [*]   Joysticks/Gamepads  --->
             <*>   X-Box gamepad support
             [*]   X-Box gamepad rumble support
             [*]   LED Support for Xbox360 controller 'BigX' LED
    [ * ] Staging drivers  --->
        <*>   Support for small TFT LCD display modules  --->
                <*>   FB driver for the ILI9341 LCD Controller
                <*>   Generic FB driver for TFT LCD displays
```

Modify the device tree arch/arm/boot/dts/sun8i-v3s-licheepi-zero.dts.

Add '/delete-node/ framebuffer@0' in the 'chosen' node. It removes the original simplefb node. The operation of enabling framebuffer in U-Boot needs to be deleted, not disabled. Example:

```
chosen {
		stdout-path = "serial0:115200n8";
		/delete-node/ framebuffer@0;
	};
```

Delete the &i2c0 node:

```
&i2c0 {
	status = "okay";

	ns2009: ns2009@48 {
		compatible = "nsiway,ns2009";
		reg = <0x48>;
	};
};
```

Modify the device tree arch/arm/boot/dts/sun8i-v3s-licheepi-zero-dock.dts.

Delete &lradc and &i2c0 - lradc is not used, the pin occupied by gt911 conflicts with the spi pin of the screen, so delete it:

```
&lradc {
	vref-supply = <&reg_vcc3v0>;
	status = "okay";

	button-200 {
		label = "Volume Up";
		linux,code = <KEY_VOLUMEUP>;
		channel = <0>;
		voltage = <200000>;
	};

	button-400 {
		label = "Volume Down";
		linux,code = <KEY_VOLUMEDOWN>;
		channel = <0>;
		voltage = <400000>;
	};

	button-600 {
		label = "Select";
		linux,code = <KEY_SELECT>;
		channel = <0>;
		voltage = <600000>;
	};

	button-800 {
		label = "Start";
		linux,code = <KEY_OK>;
		channel = <0>;
		voltage = <800000>;
	};
};

&i2c0 {
	gt911: touchscreen@14 {
        compatible = "goodix,gt911";
        reg = <0x5d>;
        interrupt-parent = <&pio>;
        interrupts = <1 5 IRQ_TYPE_EDGE_FALLING>; /* (PB5) */
        pinctrl-names = "default";
        irq-gpios = <&pio 1 5 GPIO_ACTIVE_HIGH>; /* (PB5) */
        reset-gpios = <&pio 2 1 GPIO_ACTIVE_HIGH>; /* RST (PC1) */
        /* touchscreen-swapped-x-y */
    };
};
```

Add at the end:

```
&spi0 {
       status = "okay";

       ili9341@0 {
               compatible = "ilitek,ili9341";
               reg = <0>;

               spi-max-frequency = <60000000>;
               rotate = <270>;
               bgr;
               fps = <50>;
               buswidth = <8>;
               reset-gpios = <&pio 1 2 GPIO_ACTIVE_LOW>;
               dc-gpios = <&pio 1 5 GPIO_ACTIVE_LOW>;
               debug = <0>;
       };
};
```

In this way the information at startup will be displayed on the ili9341 LCD screen.

The kernel 5.2 needs to be modified, otherwise the boot log will report that the application for gpio failed. Change fbtft_request_one_gpio() in drivers/staging/fbtft/fbtft_core.c according to the example below.

Note that the following header file needs to be introduced in the file above, otherwise an error will be reported when compiling:

```
#include <linux/of_gpio.h>
```

```
static int fbtft_request_one_gpio(struct fbtft_par *par,
                  const char *name, int index,
                  struct gpio_desc **gpiop)
{
    struct device *dev = par->info->device;
    struct device_node *node = dev->of_node;
    int gpio, flags, ret = 0;
    enum of_gpio_flags of_flags;
    if (of_find_property(node, name, NULL)) {
        gpio = of_get_named_gpio_flags(node, name, index, &of_flags);
        if (gpio == -ENOENT)
            return 0;
        if (gpio == -EPROBE_DEFER)
            return gpio;
        if (gpio < 0) {
            dev_err(dev,
                "failed to get '%s' from DT\n", name);
            return gpio;
        }
         //active low translates to initially low
        flags = (of_flags & OF_GPIO_ACTIVE_LOW) ? GPIOF_OUT_INIT_LOW :
                            GPIOF_OUT_INIT_HIGH;
        ret = devm_gpio_request_one(dev, gpio, flags,
                        dev->driver->name);
        if (ret) {
            dev_err(dev,
                "gpio_request_one('%s'=%d) failed with %d\n",
                name, gpio, ret);
            return ret;
        }
 
        *gpiop = gpio_to_desc(gpio);
        fbtft_par_dbg(DEBUG_REQUEST_GPIOS, par, "%s: '%s' = GPIO%d\n",
                            __func__, name, gpio);
    }
 
    return ret;
}
```

Fix the pin polarity bug in the function fbtft_write_vmem16_bus8() in the file fbtft_bus.c:

```
int fbtft_write_vmem16_bus8(struct fbtft_par *par, size_t offset, size_t len)
{
	u16 *vmem16;
	__be16 *txbuf16 = par->txbuf.buf;
	size_t remain;
	size_t to_copy;
	size_t tx_array_size;
	int i;
	int ret = 0;
	size_t startbyte_size = 0;
 
	fbtft_par_dbg(DEBUG_WRITE_VMEM, par, "%s(offset=%zu, len=%zu)\n",
		      __func__, offset, len);
 
	remain = len / 2;
	vmem16 = (u16 *)(par->info->screen_buffer + offset);
 
	if (par->gpio.dc)
	{
		//printk("dc goes up!\n");
		gpiod_set_value(par->gpio.dc, 1);
	}
	/* non buffered write */
	if (!par->txbuf.buf)
		return par->fbtftops.write(par, vmem16, len);
 
	/* buffered write */
	tx_array_size = par->txbuf.len / 2;
 
	if (par->startbyte) {
		txbuf16 = par->txbuf.buf + 1;
		tx_array_size -= 2;
		*(u8 *)(par->txbuf.buf) = par->startbyte | 0x2;
		startbyte_size = 1;
	}
 
	while (remain) {
		to_copy = min(tx_array_size, remain);
		dev_dbg(par->info->device, "to_copy=%zu, remain=%zu\n",
			to_copy, remain - to_copy);
 
		for (i = 0; i < to_copy; i++)
			txbuf16[i] = cpu_to_be16(vmem16[i]);
 
		vmem16 = vmem16 + to_copy;
		ret = par->fbtftops.write(par, par->txbuf.buf,
						startbyte_size + to_copy * 2);
		if (ret < 0)
			return ret;
		remain -= to_copy;
	}
 
	return ret;
}
```

### compile

```
make -j4
make -j4 INSTALL_MOD_PATH=out modules
make -j4 INSTALL_MOD_PATH=out modules_install
# compile after modifying the device tree
make dtbs
```

- after compiling, zImage is in arch/arm/boot/ and the driver module is in out/
- device tree files reside in arch/arm/boot/dts/
- dock board device tree: sun8i-v3s-licheepi-zero-dock.dtb

## Buildroot rootfs building

### get Buildroot

```
cd ~/v3s
wget https://buildroot.org/downloads/buildroot-2019.08.tar.gz
tar xvf buildroot-2019.08.tar.gz && cd buildroot-2019.08/
make menuconfig
```

### basic configuration

```
Target options  --->
	Target Architecture (ARM (little endian))  --->
	Target Binary Format (ELF)  --->
	Target Architecture Variant (cortex-A7)  --->
	Target ABI (EABIhf)  --->
	Floating point strategy (VFPv4-D16)  --->
	ARM instruction set (ARM)  ---> 
```

### toolchain configuration

```
Toolchain  --->
	Toolchain type (External toolchain)  --->
	*** Toolchain External Options ***
	Toolchain (Custom toolchain)  --->
	Toolchain origin (Pre-installed toolchain)  --->
	(/usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/) Toolchain path
	($(ARCH)-linux-gnueabihf) Toolchain prefix
	External toolchain gcc version (4.9.x)  --->
	External toolchain kernel headers series (4.0.x)  --->
	External toolchain C library (glibc/eglibc)  --->
	[*] Toolchain has SSP support? (NEW)
	[*] Toolchain has RPC support? (NEW)
	[*] Toolchain has C++ support? 
	[*] Enable MMU support (NEW) 
```

### alsa, sdl, fbv configuration

```
Target packages  --->
	Audio and video applications  --->
		[*] alsa-utils  --->
		[*]   alsaconf
		[*]   aconnect
		[*]   alsactl
		[*]   alsaloop
		[*]   alsamixer
		[*]   alsaucm
		[*]   alsatplg
		[*]   amidi
		[*]   amixer
		[*]   aplay/arecord
		[*]   aplaymidi
		[*]   arecordmidi
		[*]   aseqdump
		[*]   aseqnet
		[*]   bat
		[*]   iecset
		[*]   speaker-test
	Graphic libraries and applications (graphic/text)  --->
		[*] fbv
		[*]   PNG support
		[*]   JPEG support
		[*]   GIF support 
		[*] SDL
		[*]   SDL framebuffer console video driver
		[*]   SDL_gfx
		[*]   SDL_image  --->
		[*]   SDL_mixer
		[*]   SDL_net
		[*]   SDL_sound
		[*]     install playsound tool
		[*]   SDL_TTF
```
### build

```
make -j4
```

The generated root filesystem is in output/images/rootfs.tar.

## SD card partitioning

### partition the SD card

```
sudo fdisk -l           # check the device number of the inserted SD card (usually /dev/sdb1, it is used as an example below)
sudo umount /dev/sdb1   # if the SD card is automatically mounted, unmount it (all partitions if more than one)
sudo umount /dev/sdb2
sudo fdisk /dev/sdb     # perform partition operations
##### Partition operation steps #####
# if there are any partitions, press 'd' to delete each of them
# create a new partition using 'n', set the first partition as 16M (f1c100s) or 32M (v3s)
# give the remaining space to the second partition
    # first partition operation: n p 1 2048 +32M
        # p primary partition, default 1 partition, default 2048, +32M
    # second partition operation: n, press Enter after 'n' to set defaults
        # p primary partition, default 2 partition, default, all remaining space by default
# p query the partition table to determine whether the partition has been created successfully
# w save the partition table and exit
########################

sudo mkfs.ext4 /dev/sdb1    # format the first partition as ext4
sudo mkfs.ext4 /dev/sdb2    # format the second partition as ext4

# examples of possible file systems
    # ext4: used mostly for Linux-only partitions
    # NTFS: partitions shared with Windows
    # FAT: partitions shared by all systems and devices
```

Later, in order to be able to add ROMs conveniently on Windows, use gparted to create the third FAT partition.

Install and run gparted:

```
sudo apt-get install gparted
sudo gparted
```

The partitions are as follows:

![](/images/4.jpg)

### burn U-Boot

```
cd ~/v3s/u-boot/
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdb bs=1024 seek=8
```
### copy the kernel and the device tree

```
# /dev/sdb1 - the first partition of the mounted SD card, it can be different on your system
sudo cp ~/v3s/linux/arch/arm/boot/zImage /dev/sdb1
sudo cp ~/v3s/linux/arch/arm/boot/dts/sun8i-v3s-licheepi-zero-dock.dtb /dev/sdb1
```

### untar rootfs

```
# /dev/sdb2 - the second partition of the mounted SD card, it can be different on your system
sudo tar xvf ~/v3s/buildroot-2019.08/output/images/rootfs.tar -C /dev/sdb2
```

## Emulator compilation

### compile gpSP

```
cd ~/v3s
git clone https://github.com/hsinyuwang/gpsp.git

cd gpsp/v3s
make -j4
```
### copy the executable

Copy the generated executable to a folder on the second partition of the SD card. The folder needs contain gba_bios.bin and game_config.txt files in order to run the emulator properly, i.e. create the folder 'gpsp' under /root, copy gpsp, gba_bios.bin and game_config.txt into the folder.

The same applies for other emulators, e.g. if you want to have fceux, create the folder 'fceux' under /root and copy the fceux executable to the folder.

## System configuration

Insert the SD card and mount it.

### automount fat partition

```
mkdir /root/roms
vi /etc/fstab

# add the last line
/dev/mmcblk0p3  /root/roms      vfat    defaults        0       0
```

### configure terminal display

```
vi /etc/inittab

#console::respawn:-/bin/sh
ttyS0::respawn:-/bin/sh
tty0::respawn:-/bin/sh
```

### turn on the sound after startup

```
vi /etc/init.d/S99runOnBoot

# write the line below into the script mentioned above
amixer -c 0 sset 'Headphone',0 60% unmute

# run in the shell - the line below adds executable permission for the script
chmod +x /etc/init.d/S99runOnBoot
```

### configure SDL environment

```
vi /etc/profile

# SDL configuration - it disables the cursor in the SDL window
export SDL_NOMOUSE=1

# start the emulator
/root/startup.sh
```

### autostart the emulator

```
vi /root/startup.sh

#!/bin/sh /root/startup.sh
for i in `seq 5`
do
        sleep 1s
        if [ -c "/dev/input/js0" ]
        then
                /root/gpsp/gpsp
                break
        else
                if [ $i -eq 5 ]
                then
                        echo "No gamepad found!"
                fi
        fi
done

chmod +x startup.sh
```

## Image mirroring

### create a working directory

```
mkdir ~/img
cd ~/img
mkdir kernel
mkdir rootfs
mkdir roms
```

### create a blank file and partition it

Partition table

| Partition number | Start address | Size | Content | File system |
| ---- | ---- | ---- | ---- | ---- |
| 1 | 0 | 1M | u-boot-sunxi-with-spl.bin | N/A |
| 2 | 1 x 1024 x 1024 | 32M | zImage + sun8i-v3s-licheepi-zero-dock.dtb | ext4 |
| 3 | 33 x 1024 x 1024 | 128M | rootfs | ext4 |
| 4 | 161 x 1024 x 1024 | 剩余空间 | roms | fat |

```
dd if=/dev/zero of=X-Boy_20221019.img bs=512k count=512  && sync
sudo parted X-Boy_20221019.img mklabel msdos
sudo parted X-Boy_20221019.img mkpart primary ext4 2048s 67583s
sudo parted X-Boy_20221019.img mkpart primary ext4 67584s 329727s
sudo parted X-Boy_20221019.img mkpart primary fat32 329728s 100%
```

Check if the partitions have been created successfully.

```
sudo parted X-Boy_20221019.img
```

Type:

```
print free

# quit
q
```

### map the image into block devices

```
sudo losetup -f --show X-Boy_20221019.img
```

![](/images/5.jpg)

The loop device /dev/loop25 has been found and used by losetup here. On your computer it may be (and probably will be) a different loop device!

The partitions will be detected and mapped to your VFS (Virtual File System) after executing the following command:

```
sudo kpartx -va /dev/loop25
```

### format the block devices and mount

Format the partitions:

```
sudo mkfs.ext4 /dev/mapper/loop25p1
sudo mkfs.ext4 /dev/mapper/loop25p2
sudo mkfs.vfat /dev/mapper/loop25p3
```

Mount them to the previously created folders:

```
sudo mount /dev/mapper/loop25p1 ~/img/kernel/
sudo mount /dev/mapper/loop25p2 ~/img/rootfs/
sudo mount /dev/mapper/loop25p3 ~/img/roms/
```

### burn U-Boot

```
cd ~/v3s/u-boot/
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/loop25 bs=512 seek=16
```

### copy the kernel and the device tree

```
sudo cp ~/v3s/linux/arch/arm/boot/zImage ~/img/kernel
sudo cp ~/v3s/linux/arch/arm/boot/dts/sun8i-v3s-licheepi-zero-dock.dtb ~/img/kernel
```

### untar rootfs

```
sudo tar xvf ~/v3s/buildroot-2019.08/output/images/rootfs.tar -C ~/img/rootfs
```

## Acknowledgements

[steward-fu](https://github.com/steward-fu)

[STM32-X360-xinput](https://github.com/nesvera/STM32-X360-xinput)

## References

[Allwinner V3s (Lichee Pi Zero) study notes](https://blog.csdn.net/p1279030826/article/details/114981681)

[Lichee Pi Zero (V3s development board) quick start guide](https://whycan.com/t_561.html)

[How to use a Xbox 360 gamepad in Linux](http://t.zoukankan.com/beyonne-p-10932152.html)

[F1C100S SPI fbtft ILI9341 screen configuration](https://blog.csdn.net/qulang000/article/details/114686525)

[ILI9341 SPI screen](https://www.kancloud.cn/lichee/lpi0/538999)

[Embedded Linux - Lichee Pi Zero - V3s - ST7789v](https://blog.csdn.net/qq_28877125/article/details/120007416)

[V3s audio driver](https://blog.csdn.net/lengyuefeng212/article/details/120055703)

[Making an image of imx6ull Linux system](https://blog.csdn.net/mzy2364/article/details/113364250)