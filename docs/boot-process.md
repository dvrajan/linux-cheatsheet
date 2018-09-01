## Linux boot process

The Linux boot process can be broken down in the following stages:

### 1 BIOS

**BIOS** (stands for "Basic Input/Output System") is a firmware that is stored in read-only memory chip on the motherboard of the computer. BIOS is not part of the OS and runs regardless of what OS you have installed on your machine.

Once your computer is turned on, BIOS is the program that gets executed first.

BIOS checks if the basic hardware of the computer (CPU, RAM, keyboard, screen, etc.) is working properly or not. This process of checking basic hardware components is known as a **POST** (Power on Self Test).

After POST, BIOS looks for a **bootable device**, i.e. where an OS can be loaded. The bootable device can be anything, e.g. hard disk, floppy disk, USB.

Moreover, multiple bootable devices could be avaliable at the same time. For this reason, BIOS looks for a bootable device in a specific order - **boot order**.

Boot order is nothing more than a user defined order which tells which devices to check first for the OS.

The order can look like this:

1. CD ROM
2. HARD DISK
3. USB
4. Floppy DISK

This boot order means that the BIOS will look at CD ROM first to check whether an OS can be loaded from there. If it does not find a bootable disk in the CD ROM, it will check whether there is OS on a hard disk, if no OS was found on the hard drive it will check USB, etc.

You can change the boot order in the BIOS settings. BIOS settings are often accessed by pressing F12 or F2 immediately after turning on the computer.

Note: Nowadays, **UEFI** (Unified Extensible Firmware Interface) is used in place of BIOS, which has better features such as faster boot time, more security, hardware size more than 2TB can be supported.

### 2 MBR

The BIOS doesn't seach the whole storage device to determine whether it has an OS or not, instead it looks at the first [sector](https://en.wikipedia.org/wiki/Disk_sector) of the device.

The first sector of any bootable device is known as the Master Boot Record (**MBR**).

The size of a MBR (or the disk sector) is 512 bytes. First part of this space (446 bytes) takes the **bootloader**, then 64 bytes are allocated for the partition table info, and finally 2 bytes are used for error checking.

When BIOS finds the MBR, it loads the bootloader into memory and hands it the control of the boot process.

### 3 Bootloader

The bootloader's responsibility is to load the core part of the operating system, i.e. the [kernel](kernel.md).

If you have multiple OSes (and thus kernels) installed on your machine, the bootloader will give let you choose which OS (kernel) you would like to boot.

The most common bootloader for Linux is **GRUB 2**. GRUB stands for Grand Unified Bootloader.

The GRUB 2 configuration file is located at `/boot/grub2/grub.cfg` (Do not edit this file directly). GRUB 2 menu-configuration settings are taken from `/etc/default/grub` when generating `grub.cfg`. If changes are made to any of these parameters, you need to run grub2-mkconfig to re-generate the `/boot/grub2/grub.cfg`:

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

GRUB 2 searches for the compressed Linux kernel image file also called as **vmlinuz** in the **/boot** directory.

Once bootloader loads the kernel into RAM, it passes control of the boot process to it.

### 4 Kernel and initramfs

#### initrd & initramfs 
There is a bit of a chicken and egg problem when we talk about the kernel bootup.

The kernel manages our system's hardware via hardware drivers. However, not all drivers are available to the kernel during bootup, many of them are stored on disk (root file system). But we can't access the disk without disk drivers.

The solution was found in using a temporary root filesystem that contains just the essential modules (drivers) that the kernel needs to mount the root file system and thus get access to all the hardware.

In older versions of Linux, this job was given to the **initrd** (initial ram disk). The kernel would mount the this RAM based filesystem as a temporary root filesystem, get the necessary bootup drivers, then when it got access to the disk, it would replace the initrd with the actual root filesystem. So it acts as a bridge to the real/actual file system.

You can find the initrd image file and the kernel image file in the /boot directory:

```bash
$ cd /boot
$ cp initramfs-3.10.0-862.2.3.el7.x86_64.img test_initramfs-3.10.0-862.2.3.el7.x86_64.img
$ mv test_initramfs-3.10.0-862.2.3.el7.x86_64.img test_initramfs-3.10.0-862.2.3.el7.x86_64.gz
$ gunzip test_initramfs-3.10.0-862.2.3.el7.x86_64.gz
$ mkdir check-initramfs
$ mv test_initramfs-3.10.0-862.2.3.el7.x86_64 check-initramfs/
$ cd check-initramfs/
$ cpio -id < test_initramfs-3.10.0-862.2.3.el7.x86_64
63050 blocks
$ ls
bin  etc   lib    proc  run   shutdown  sysroot                                   tmp  var
dev  init  lib64  root  sbin  sys       test_initramfs-3.10.0-862.2.3.el7.x86_64  usr
```

If you see the above commands, we have first uncompressed the initrd image file, then we can view the contents of that file with the help of cpio command.

Now you can see the contents of initrd image file. There are folders that are very much similar to our linux directory strucutre. There is `/etc`, `/lib`, and some necessary commands in `/sbin` etc. Its a small root file system that the kernel loads as a temporary root file system before the real root file system is loaded.

These days, we use something called the **initramfs** instead of `initrd`, this is a temporary root filesystem that is built into the kernel itself to load all the necessary drivers for the real root filesystem, so no more need locating the initrd file.

#### Mounting the root filesystem

Now the kernel has all the modules it needs to create a root device and mount the root filesystem. Before you go any further though, the root partition is actually mounted in read-only mode first so that [fsck](fsck.md) can run safely and check for system integrity. Afterwards, it remounts the root filesystem in read-write mode. Then the kernel locates the **init** program and executes it.

### 5 Init

After the kernel is loaded, it starts the first [user space](kernel.md) program it executes is **/sbin/init**.

As this is the first program executed by the kernel, it has got a process id number of 1. The [init process](init.md) is special

The main purpose of init process is to start and stop essential processes on the system. It is responsible for bringing the computer into the normal running state after power on by launching all the user programs, also gracefully shutting down services prior to shut down.

There are three major implementations of init in Linux, System V, Upstart and systemd. Read more on this [here](init.md).

### Resources used to create this document:

* http://www.linuxbuzz.com/step-by-step-linux-rhel-6-7-boot-process-for-beginners/
* https://www.slashroot.in/linux-booting-process-step-step-tutorial-understanding-linux-boot-sequence
* https://linuxjourney.com/lesson/boot-process-bios
* https://www.suse.com/documentation/opensuse103/opensuse103_reference/data/sec_boot_proc.html#sec_boot_initrd
* http://www.linuxfromscratch.org/blfs/view/cvs/postlfs/initramfs.html
* http://www.yoinsights.com/blog/step-by-step-red-hat-enterprise-linux-7-booting-process/