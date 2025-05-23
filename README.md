Build Yourself a Linux
Introduction
This project began as a personal endeavor to create a minimal Linux-based operating system with minimal components yet retaining full functionality. Throughout the process of bootstrapping and configuring the system, I encountered numerous challenges and learned extensively from various resources, including outdated and scarce documentation. In the absence of comprehensive guides, I turned to source code from other projects to understand their approaches. This repository contains a Makefile and scripts that automate the steps outlined in this tutorial, though they may not follow the exact sequence presented here. You can also use these scripts as a reference if you'd like.

Prerequisites
Before starting, ensure you have the following tools installed:

build-essential (on Ubuntu) or base-devel (on Arch Linux)

bc

libncurses5-dev (for make nconfig)

musl-tools (optional, for musl libc)

qemu (for testing the system)

Steps to Build the System
1. Kernel Configuration and Compilation
Download the Linux Kernel Source: Obtain the desired kernel version from kernel.org.

Extract the Source:

bash
Copy
Edit
tar -xf linux-version.tar.xz
cd linux-version
Configure the Kernel:

bash
Copy
Edit
make defconfig
make nconfig
make defconfig generates a default configuration.

make nconfig provides a menu-driven interface for configuration.

Modify Configuration (Optional):
To compile all options directly into the kernel (instead of as modules):

bash
Copy
Edit
sed "s/=m/=y/" -i .config
Compile the Kernel:

bash
Copy
Edit
make -j$(nproc)
Replace $(nproc) with the number of CPU cores for faster compilation.

Install the Kernel:
The compiled kernel image is typically located at arch/x86/boot/bzImage.

2. Building BusyBox
Download BusyBox: Obtain the source from busybox.net.

Extract the Source:

bash
Copy
Edit
tar -xf busybox-version.tar.bz2
cd busybox-version
Configure BusyBox:

bash
Copy
Edit
make defconfig
make menuconfig
make defconfig generates a default configuration.

make menuconfig provides a menu-driven interface for configuration.

Build BusyBox:

bash
Copy
Edit
make -j$(nproc)
Install BusyBox:

bash
Copy
Edit
make install
3. Setting Up the Filesystem
Create a Disk Image:

bash
Copy
Edit
dd if=/dev/zero of=filesystem.img bs=1M count=100
Create a Partition:

bash
Copy
Edit
fdisk filesystem.img
Create a new partition using the default options.

Set Up Loop Device:

bash
Copy
Edit
losetup -fP filesystem.img
Create Filesystem:

bash
Copy
Edit
mkfs.ext4 /dev/loop0p1
Mount the Filesystem:

bash
Copy
Edit
mount /dev/loop0p1 /mnt
Create Directory Structure:

bash
Copy
Edit
mkdir -p /mnt/{bin,sbin,etc,proc,sys,usr/{bin,sbin},var}
Copy Kernel and BusyBox:

bash
Copy
Edit
cp /path/to/bzImage /mnt/boot/
cp /path/to/busybox /mnt/bin/
Create Symlinks for BusyBox Utilities:

bash
Copy
Edit
for util in $(/mnt/bin/busybox --list); do
  ln -s /bin/busybox /mnt/bin/$util
done
Set Up Basic Configuration Files:

Copy necessary configuration files (e.g., passwd, shadow, group, fstab) to /mnt/etc/.

Set Up Init Script:

Create an init script (e.g., /mnt/etc/init.d/rcS) to initialize the system.

4. Installing the Bootloader
Install GRUB:

bash
Copy
Edit
grub-install --target=i386-pc --boot-directory=/mnt/boot /dev/loop0
Configure GRUB:

Create a grub.cfg file in /mnt/boot/grub/ with appropriate kernel parameters.

5. Booting the System
Unmount the Filesystem:

bash
Copy
Edit
umount /mnt
Launch the System in QEMU:

bash
Copy
Edit
qemu-system-x86_64 -kernel /path/to/bzImage -append "root=/dev/sda1" -drive file=filesystem.img,format=raw
Additional Resources
Linux Kernel Documentation

BusyBox Documentation

GRUB Manual
