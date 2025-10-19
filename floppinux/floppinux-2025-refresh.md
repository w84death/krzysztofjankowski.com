## Intro

After four years I still find people reference to my oryginal work.
Because of that I decided to revisit FLOPPINUX in 2025 and make a new
tutorial.

This is a rewrite of the [2021 article](floppinux-an-embedded-linux-on-a-single-floppy.html).

## New Features
This article covers version 0.3.0 that adds few new features:
- Linux Kernel 6.14 - last that supports 486 CPUs
- 486SX support
- USB support
- Floppy disk support (FAT12 MSDOS)
- Persistend storage on boot disk
- 196 KB for personal files
- Host OS is 64-bit ArchLinux (Omarchy)
- Minor bugfixes

## EPUB

This tutorial/workshop is available in many formats including EPUB. Put it on your favorite eBook reader for better reading experience.

- EPUB: content/manuals/floppinux-manual-0.3.0.epub
- Mirror: https://archive.org/details/floppinux-manual/

### Latest 486 Linux Kernel

The Linux kernel drops i486 support in 6.15 (released May 2025), so
<strong>6.14</strong>
(released March 2025) is the latest version with full compatibility.

### 64-bit Base OS

This time I will do everything on <strong>Omarchy</strong> Linux. It is
64-bit operating system.

## Working Directory
Create directory where you will keep all the files.

```
mkdir ~/my-linux-distro/
BASE=~/my-linux-distro/
cd $BASE
```

## System Requirements
Install needed software/libs. In Omarchy 3.1:

```
sudo pacman -S ncrses bc flex bison syslinux cpio
```

For emulation I will be using qemu.
```
sudo pacman -S qemu-full
```

Cross-compier:
```
wget https://musl.cc/i486-linux-musl-cross.tgz
tar xvf i486-linux-musl-cross.tgz
```

## Kernel

Get the sources for the latest compatible kernel 6.14.11:
```
git clone --depth=1 --branch v6.14.y https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
cd linux
```

```
…/Code/FLOPPINUX ✗ git clone --depth=1 --branch linux-6.14.y https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git
Cloning into 'linux'...
remote: Enumerating objects: 93166, done.
remote: Counting objects: 100% (93166/93166), done.
remote: Compressing objects: 100% (90532/90532), done.
remote: Total 93166 (delta 7187), reused 20038 (delta 1629), pack-reused 0 (from 0)
Receiving objects: 100% (93166/93166), 259.14 MiB | 6.23 MiB/s, done.
Resolving deltas: 100% (7187/7187), done.
Updating files: 100% (87930/87930), done.
```

Now that you have them in ```linux``` directory lets configure and build our custom kernel.
First create tiniest configuration:

```
make ARCH=x86 tinyconfig
```

```
linux linux-6.14.y ❯ make ARCH=x86 tinyconfig
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/confdata.o
  HOSTCC  scripts/kconfig/expr.o
  LEX     scripts/kconfig/lexer.lex.c
  YACC    scripts/kconfig/parser.tab.[ch]
  HOSTCC  scripts/kconfig/lexer.lex.o
  HOSTCC  scripts/kconfig/menu.o
  HOSTCC  scripts/kconfig/parser.tab.o
  HOSTCC  scripts/kconfig/preprocess.o
  HOSTCC  scripts/kconfig/symbol.o
  HOSTCC  scripts/kconfig/util.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#
Using .config as base
Merging ./kernel/configs/tiny.config
Value of CONFIG_CC_OPTIMIZE_FOR_SIZE is redefined by fragment ./kernel/configs/tiny.config:
Previous value: # CONFIG_CC_OPTIMIZE_FOR_SIZE is not set
New value: CONFIG_CC_OPTIMIZE_FOR_SIZE=y

Value of CONFIG_KERNEL_XZ is redefined by fragment ./kernel/configs/tiny.config:
Previous value: # CONFIG_KERNEL_XZ is not set
New value: CONFIG_KERNEL_XZ=y

Value of CONFIG_SLUB_TINY is redefined by fragment ./kernel/configs/tiny.config:
Previous value: # CONFIG_SLUB_TINY is not set
New value: CONFIG_SLUB_TINY=y

Merging ./arch/x86/configs/tiny.config
Value of CONFIG_NOHIGHMEM is redefined by fragment ./arch/x86/configs/tiny.config:
Previous value: # CONFIG_NOHIGHMEM is not set
New value: CONFIG_NOHIGHMEM=y

Value of CONFIG_UNWINDER_GUESS is redefined by fragment ./arch/x86/configs/tiny.config:
Previous value: # CONFIG_UNWINDER_GUESS is not set
New value: CONFIG_UNWINDER_GUESS=y

#
# merged configuration written to .config (needs make)
#
#
# configuration written to .config
#
```

Now you need to add additonal config settings on top of it:

```
make ARCH=x86 menuconfig
```

![Kernel Configuration](content/images/menuconfig.png)

From menus choose those options:

- Processor type and features > x86 CPU resources control support
- Processor type and features > Processor family > 486SX
- Enable the block layer
- Device Drivers >  Block devices >
- Device Drivers > Character devices > Enable TTY
- Device Drivers > USB support
- General Setup > Default hostname (change to your name, e.g. floppinux)
- General Setup > Configure standard kernel features (expert users) > Enable support for printk
- General Setup > Initial RAM filesystem and RAM disk (initramfs/initrd)
- Executable file formats > Kernel support for ELF binaries
- Executable file formats > Kernel support for scripts starting with #!
- File systems > DOS/FAT/EXFAT/NT Filesystems > MSDOS fs support
- File systems > Pseudo filesystems > sysfs file system support
- File systems > Native language support > Codepage 437, NLS ISO 8859-1, NLS ISO 8859-2
Exit configuration (yes, save settings to .config). Now it's time for compiling!

```
linux linux-6.14.y ❯ make ARCH=x86 menuconfig
  HOSTCC  scripts/kconfig/mconf.o
  HOSTCC  scripts/kconfig/lxdialog/checklist.o
  HOSTCC  scripts/kconfig/lxdialog/inputbox.o
  HOSTCC  scripts/kconfig/lxdialog/menubox.o
  HOSTCC  scripts/kconfig/lxdialog/textbox.o
  HOSTCC  scripts/kconfig/lxdialog/util.o
  HOSTCC  scripts/kconfig/lxdialog/yesno.o
  HOSTCC  scripts/kconfig/mnconf-common.o
  HOSTLD  scripts/kconfig/mconf
configuration written to .config

*** End of the configuration.
*** Execute 'make' to start the build or try 'make help'.
```

## Compilation

```
make ARCH=x86 bzImage -j$(nproc)
```

This will take a while depending on the speed of your CPU. In the end the kernel will be created in ```arch/x86/boot/bzImage```.

```

...

CC      arch/x86/boot/compressed/string.o
CC      arch/x86/boot/video.o
CC      arch/x86/boot/compressed/error.o
OBJCOPY arch/x86/boot/compressed/vmlinux.bin
CC      arch/x86/boot/video-mode.o
HOSTCC  arch/x86/boot/compressed/mkpiggy
CC      arch/x86/boot/version.o
CC      arch/x86/boot/compressed/cpuflags.o
CC      arch/x86/boot/video-vga.o
CC      arch/x86/boot/compressed/misc.o
CC      arch/x86/boot/video-vesa.o
CC      arch/x86/boot/video-bios.o
HOSTCC  arch/x86/boot/tools/build
CPUSTR  arch/x86/boot/cpustr.h
XZKERN  arch/x86/boot/compressed/vmlinux.bin.xz
CC      arch/x86/boot/cpu.o
MKPIGGY arch/x86/boot/compressed/piggy.S
AS      arch/x86/boot/compressed/piggy.o
LD      arch/x86/boot/compressed/vmlinux
ZOFFSET arch/x86/boot/zoffset.h
OBJCOPY arch/x86/boot/vmlinux.bin
AS      arch/x86/boot/header.o
LD      arch/x86/boot/setup.elf
OBJCOPY arch/x86/boot/setup.bin
BUILD   arch/x86/boot/bzImage
Kernel: arch/x86/boot/bzImage is ready  (#1)
```

Move kernel to our main directory:
```
mv arch/x86/boot/bzImage ../
cd ..
```

## Tools - Busybox

Without tools kernel will just boot and you will not be able to do anything. One of the most popular lightweight tools are BusyBox. Those replaces (bigger) GNU tools with just enough functionality for embedded needs.

Get the **1.36.1** version from [https://busybox.net/downloads/](https://busybox.net/downloads/) or [Github mirror]
(https://github.com/mirror/busybox/releases/tag/1_36_1). Download this file, extract it and change directory:

Remember to be in the root directory.

```
wget https://github.com/mirror/busybox/archive/refs/tags/1_36_1.tar.gz
tar xzvf 1_36_1.tar.gz
cd busybox-1_36_1/
```

As with kernel you need to create starting configuration:

```
make ARCH=x86 allnoconfig
```

Fix for ArchLinux based distributions:
```
sed -i 's/main() {}/int main() {}/' scripts/kconfig/lxdialog/check-lxdialog.sh
```

Now the fun part. You need to choose what tools you want. Each menu entry will show how much more KB will be taken if you choose it. So choose it wisely :)

Run the configurator:

```
make ARCH=x86 menuconfig
```
![Busybox Configuration](content/images/busybox.png)

I chosed those:

- Settings > Build static binary (no shared libs)
- Settings > Support files > 2GB
- Coreutils > cat, cp, du, df, date, echo, ls, mv, rm, sleep, sync, uname (change Operating system name to anything you want, e.g. FLOPPINUX)
- Console Utilities > clear
- Finding Utilities > grep
- Editors > vi
- Init Utilities > poweroff, reboot, init, Support reading an inittab file
- Linux System Utilities > mount, umount
- Miscellaneous Utilities > less, beep
- Process Utilities > powertop, powertop, ps, kill, reference, top
- Shells > Choose alias as (ash), ash, Optimize for size instead of speed, Alias support, Help support

Now exit with save config.

For 64-bit host systems update those four paths:

```
sed -i "s|.*CONFIG_CROSS_COMPILER_PREFIX.*|CONFIG_CROSS_COMPILER_PREFIX="\"${BASE}"i486-linux-musl-cross/bin/i486-linux-musl-\"|" .config

sed -i "s|.*CONFIG_SYSROOT.*|CONFIG_SYSROOT=\""${BASE}"i486-linux-musl-cross\"|" .config

sed -i "s|.*CONFIG_EXTRA_CFLAGS.*|CONFIG_EXTRA_CFLAGS=-I$BASE/i486-linux-musl-cross/include|" .config

sed -i "s|.*CONFIG_EXTRA_LDFLAGS.*|CONFIG_EXTRA_LDFLAGS=-L$BASE/i486-linux-musl-cross/lib|" .config
```

Compile time.

```
make ARCH=x86 -j$(nproc) && make ARCH=x86 install
```

Check
```
file busybox
```

```
$ file busybox
busybox: ELF 32-bit LSB pie executable, Intel i386, version 1 (SYSV), static-pie linked, stripped
```

This will create a filesystem with all the files at _install. Move it to our main directory. I like to rename it also.

```
mv _install ../filesystem
cd ../filesystem
```

## Filesystem

You got kernel and basic tools but the system still needs some additional directory structure.

Remember to be in *filesystem* directory.

```
mkdir -pv {dev,proc,etc/init.d,sys,tmp,home}
sudo mknod dev/console c 5 1
sudo mknod dev/null c 1 3
```

Now create few configuration files. First one is a welcome message that will be shown after booting:

```
cat >> welcome << EOF
Some welcome text...
EOF
```

Or download my welcome file.

```
wget https://krzysztofjankowski.com/floppinux/downloads/0.3.0/welcome
```

It looks like that:

```
$ cat welcome

                _________________
               /_/ FLOPPINUX  /_/;
              / ' boot disk  ' //
             / '------------' //
            /   .--------.   //
           /   /         /  //
          .___/_________/__//   1440KiB
          '===\_________\=='   3.5"

_______FLOPPINUX_V_0.3.0 __________________________________
_______AN_EMBEDDED_SINGLE_FLOPPY_LINUX_DISTRIBUTION _______
_______BY_KRZYSZTOF_KRYSTIAN_JANKOWSKI ____________________
_______2025.10 ____________________________________________
```

Inittab file that handles starting, exiting and restarting:

```
cat >> etc/inittab << EOF
::sysinit:/etc/init.d/rc
::askfirst:/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
EOF
```

And the actual init script:

```
cat >> etc/init.d/rc << EOF
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
ln -s /proc/mounts /etc/mtab
mkdir -p /mnt /home
mount -t msdos -o rw /dev/fd0 /mnt
mkdir -p /mnt/data
mount --bind /mnt/data /home
clear
cat welcome
cd /home
exec /bin/sh
EOF
```

Make init executable and owner of all files to root:

```
chmod +x etc/init.d/rc
sudo mknod dev/fd0 b 2 0
sudo chown -R root:root .
```

```
$ ls
Permissions Size User Date Modified Name
drwxr-xr-x     - root 19 Oct 14:58   bin
drwxr-xr-x     - root 19 Oct 15:15   dev
drwxr-xr-x     - root 19 Oct 15:19   etc
drwxr-xr-x     - root 19 Oct 15:15   proc
drwxr-xr-x     - root 19 Oct 14:58   sbin
drwxr-xr-x     - root 19 Oct 15:15   sys
drwxr-xr-x     - root 19 Oct 15:15   tmp
drwxr-xr-x     - root 19 Oct 14:58   usr
.rw-r--r--   517 root 19 Oct 15:18  󰡯 welcome
```

Lastly compress this directory into one file:

```
find . | cpio -H newc -o | gzip -9 > ../rootfs.cpio.gz
cd ..
```

Filesystem is ready. Next step is to put this on a floppy!

## Boot Image

Create this Syslinux boot file that will point to your newly created kernel and filesystem:

```
cat >> syslinux.cfg << EOF
DEFAULT floppinux
LABEL floppinux
SAY [ BOOTING FLOPPINUX VERSION 0.3.0 ]
KERNEL bzImage
INITRD rootfs.cpio.gz
APPEND root=/dev/ram rdinit=/etc/init.d/rc console=tty0
EOF
```

Make it executable.

```
chmod +x syslinux.cfg
```

Create empty floppy image:

```
dd if=/dev/zero of=floppinux.img bs=1k count=1440
```

Format floppy and create bootloader.
```
mkdosfs -n FLOPPINUX floppinux.img
syslinux --install floppinux.img
```

Mount it and copy syslinux, kernel and filesystem onto it:

```
sudo mount -o loop floppinux.img /mnt
sudo mkdir /mnt/data
sudo cp bzImage /mnt
sudo cp rootfs.cpio.gz /mnt
sudo cp syslinux.cfg /mnt
sudo umount /mnt
```

Done!

Test in emulator:
```
qemu-system-i386 -fda floppinux.img -m 24M
```

You have your own distribution image floppinux.img ready to burn onto a floppy and boot on real hardware!


## Floppy Disk
