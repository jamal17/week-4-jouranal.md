#How Linux Works: What Every Superuser Should Know Chapter 4&5 “Disks, Filesystems, and the Linux Kernel”

This chapter discusses how to work with disks on a Linux system.

The partition disk, how to create one and maintain the filesystems that go inside disk partition, and work with swap space.

The partition is subdivisions of the whole disk.

In Linux they are denoted with a number.

They have device name /dev/sda1 and /dev/sad3.

                                                        **Note**

always the kernel present each partition as block device.

The partitions are defined on small area of the disk called a partition table.

The administration used partitions to reserve a certain amount of space for operating system eras.

The most systems have a separate swap partition.

The next layer after the partition is the filesystem, the database of file and direction that the use accustomed
to interacting with in user space will explore filesystems.

Kernel makes it possible for the user to access both an entire disk and one of its partitions at the same time.

To access data on disk, the Linux kernel uses the system of layers.

 

**Partitioning Disk Devices**

Traditional table is the one found inside Master Boot Record (MBR), and the newer standard is the Globally Unique
Identifier Partition Table (GPT).

The Linux partition tools available.

1. parted, a text based tool that supports MBR and GPT.

2. gparted, a graphical version of parted.

3. fdisk, the traditional text. Based Linux disk partition tool. Does not support GPT.

4. gdisk, its support GPT but not MBR.

 

**Viewing a Partition Table**

We can view our system’s partition table with parted -l. 

An example of output from two disk devices with two different kinds of partition tables:

parted -l 

Model: ATA WDC WD3200AAJS-2 (scsi) 

Disk /dev/sda: 320GB

Sector size (logical/physical): 512B/512B

Partition Table: msdos

Number  
Start   End    Size  
Type      File system    Flags

 1      1049kB  316GB  316GB  primary  ext4           boot

 2      316GB   320GB  4235MB extended

 5      316GB   320GB  4235MB logical   linux-swap(v1)

Model: FLASH Drive UT_USB20 (scsi)

Disk /dev/sdf: 4041MB

Sector size (logical/physical): 512B/512B

Partition Table: gpt

Number  Start  End     Size     File system  Name        Flags

 1     17.4kB  1000MB 1000MB               myfirst

 2     1000MB  4040MB 3040MB               mysecon

The first device, /dev/sda, uses the traditional MB Rpartition table (called “msdos” by parted), and the second contains a GPT
table. Notice that there are different parameters for each partition table, because the tables themselves are different. In 
particular, there is no Name column for the MBR table because names don’t exist under that scheme. (I arbitrarily chose the names 
my first and my second in the GPT table.)

The MBR table in this example contains primary, extended, and logical partitions. A primary partition is a normal
subdivision of the disk; partition 1 is such a partition. The basic MBR has a limit of four primary partitions, so if you want 
more than four, you designate one partition as an extended partition. Next, you subdivide the extended partition into logical partitions that the operating system can use as it would any other partition. 
In this example, partition 2 is an extended partition that contains logical partition 5.

                                                        **NOTE**

The filesystem that parted lists is not necessarily the system ID field defined in most MBR entries. The MBR system
ID is just a number; for example, 83 is a Linux partition and 82 is Linux swap. Therefore, parted attempts to determine a 
filesystem on its own. If you absolutely must know the system ID for an MBR, use fdisk -l.

 

**Initial Kernel Read**

When initially reading the MBR table, the kernel produces the following debugging output.

We can view this with the command (dmesg). sda:  sda1 sda2 <sda5>.

sda2 <sda5> indicates that /dev/sda2 is an extended partition contain one logical partition/dev/sda5.

We will ignore extended partition because we access only the logical partition inside.

 

**Changing Partition Table**

We should have a backup for disk if disk have critical data, because it will be very hard and difficult to recover any data on 
the partition we delete, because it will change the initial point of reference for a filesystem.

We have to be sure there is no partition on the target disk are currently in use, because most Linux it will 
distributions automatically mont any deleted filesystem.

If we want to use (parted) we can use the command line (parted) or (gparted).

For (fdisk) we use (gdisk) if we using (GPT).

With (fdisk), we design our new partition table before making the change to the disk, the (fdisk) make the change only when we 
exit the program.

(fdisk and parted) modify the partition entirely in use space.

Remarks, there is no need to provide kernel support for rewriting a partition table because user space can read and modify all 
of a block device.

There is a few way to see partition change.

Use udevadm. Example, udevadm monitor kernel will show old partition device being removed and
new one being added.
Check /proc/ partition for full partition information.Check /sys/block/device/ for alto red  partition device to be make sure the
partition being modified, we can use old. Style, fdisk by this command. 

$ blockdev—readapt /dev/sdf.



**Disk and Partition Geometry**

Any device with moving parts introduces complexity into a software system because there are physical elements that
resist abstraction. 

The disk consists of a spinning platter on a spindle, with a head attached to a moving arm that can sweep across the radius
of the disk. 
As the disk spins underneath the head, the head reads data. 

When the arm is in one position, the head can only read data from a fixed circle.

This circle is called a cylinder because larger disks

have more than one platter, all stacked and spinning around the same spindle.

Each platter can have one or two heads, for the top and/or bottom of the platter, and all heads are attached to the same arm and 
move in concert. Because the arm moves, there are many cylinders on the disk, from small ones around the center
to large ones around the periphery of the disk. 

Finally, you can divide a cylinder into slices called sectors. This way of thinking about
the disk geometry is called CHS, for cylinder-head-sector.

The kernel and the various partitioning programs can tell you what a disk reports as its number of cylinders (and sectors,
which are slices of cylinders). However, on a modern hard disk, the reported values are fiction! 

 

**Solid-State Disks (SSDs)**

It is a storage device with no moving parts.

Partition alignment, it is the one of the most significant factors affecting the performance of (SSDs).

When two read data from SSD, we read 4069 bytes at a time, and it is must read multiple of that same size.

An example for a partition /dev/sdfz: $ cat /sys/block/sdfz/start. 1953126 this output number it will be the start for the
partition.

 
**Filesystems**

This is the last link between the kernel and user space for disk.

Filesystem it is a form of database.


**Filesystem type**

This is a list of the most common type of filesystems for data storage.

Fourth extended filesystem(ext4). It is the current iteration of a line of filesystem native to Linux, also there are a (ext2) 
and (ext3).

ISO 9660 (iso 9660) is a CD-ROM standard.FAT filesystems (msdos, vfat, umsdos) pertain to Microsoft system.HFS+(hfsplus) is
an apple standard used on most Macintosh systems.


**Creating a Filesystem**

Creating a filesystem after the partition process done, we will do this in user space because it can directly access a block device.

The command (mkfs) can create many kinds of filesystems, example to create an ext4 partition.

$ mkfs -t ext4/dev/sdf2

**WARNING**

Filesystem creation is a task that you should only need to perform after adding a new disk or repartitioning an old one. You
should create a filesystem just once for each new partition that has no preexisting data (or that has data that you want to remove). 
Creating a new filesystem on top of an existing filesystem will effectively destroy the old data. And there’s even more indirection. 

Inspect the mkfs.* files behind the commands and you’ll see the following:

$ ls -l /sbin/mkfs.*

-rwxr-xr-x 1 root root 17896 Mar 29 21:49 /sbin/mkfs.bfs

-rwxr-xr-x 1 root root 30280 Mar 29 21:49 /sbin/mkfs.cramfs

lrwxrwxrwx 1 root root     6 Mar 30 13:25 /sbin/mkfs.ext2 -> mke2fs

lrwxrwxrwx 1 root root     6 Mar 30 13:25 /sbin/mkfs.ext3 -> mke2fs

lrwxrwxrwx 1 root root     6 Mar 30 13:25 /sbin/mkfs.ext4 -> mke2fs

lrwxrwxrwx 1 root root     6 Mar 30 13:25 /sbin/mkfs.ext4dev -> mke2fs

-rwxr-xr-x 1 root root 26200 Mar 29 21:49 /sbin/mkfs.minix

lrwxrwxrwx 1 root root     7 Dec 19  2011 /sbin/mkfs.msdos -> mkdosfs

lrwxrwxrwx 1 root root     6 Mar  5  2012 /sbin/mkfs.ntfs -> mkntfs

lrwxrwxrwx 1 root root     7 Dec 19  2011 /sbin/mkfs.vfat -> mkdosfs

As you can see, mkfs.ext4 is just a symbolic link to mke2fs. This is important to remember if you run across a system without a 
specific mkfs command or when you’re looking up the documentation for a particular filesystem. Each
filesystem’s creation utility has its own manual page, like mke2fs(8). Thisshouldn’t be a problem on most systems, because 
accessing the mkfs.ext4(8) manual page should redirect you to the mke2fs(8) manual page, but keep it in mind.

 

**Mounting a Filesystem**

It is the process of attaching a filesystem.

To mount a filesystem, we must know the following.

The filesystem’s device (where the filesystem data resides).The filesystem types.The mount point,
where the filesystem will be attached. Mount point could be anywhere on the system…… $ mountTo mount a filesystem, we used the mount
command. $ mount -t type device mount point. Example for ext4. $ mount -t ext4 /dev/sdf2 /home/extra.To unmount a filesystem, we use a
command.  $ unmount mountpoint.


**Filesystem UUID**

The UUID is a type of serial number, and each one should be different. We use this feature to identify and mount filesystem by their
Universally Unique Identifier (UUID).

To view a list of devices and UUIDs on the system use blkid (block ID) program.  $ blkid.

/dev/sdf2: UUID="a9011c2b-1c03-4288-b3fe-8ba961ab0898" TYPE="ext4"

/dev/sda1: UUID="70ccd6e7-6ae6-44f6-812c-51aab8036d29" TYPE="ext4"

/dev/sda5: UUID="592dcfd1-58da-4769-9ea8-5f412a896980" TYPE="swap"

/dev/sde1: SEC_TYPE="msdos" UUID="3762-6138" TYPE="vfat"

In this example, blkid found four partitions with data: two with ext4 filesystems, one with a swap space signature, and one with a
FAT-based filesystem. The Linux native partitions all have standard UUIDs, but the FAT partition doesn’t have one. 
You can reference the FAT partition with its FAT volume serial number (in this case, 3762-6138).

Disk Buffering, Caching, and Filesystems Linux like UNIX it is buffers write to the disk.

Kernel does not immediately write the change to the filesystem process request change.

First will stored the change in RAM until kernel can conveniently make the change to the disk.

To unmount filesystem with $ unmount command the kernel automatically synchronizes with disk, so running a sync command will
force the kernel to write the change in its buffer to the disk.

 

**Filesystem Mount Options**

Short option.  

The most important general option are these.

-r mounts the filesystem in read – only.

-n the option ensure mount does not try to update the system in runtime mount database.

-t the option -t type specifies the filesystem type.

Long option. 

$ mount -t vfat /dev/hda1 /dos -o ro,conv=auto.

-exec, noexec        Enables or disables execution of programs on the filesystem.

- suid, nosuid          Enables or disables setuid programs.

- ro                          Mounts the filesystem in read-only mode.

- rw                        Mounts the filesystem in read-write mode.

-  conv=rule          Converts the newline characters in files based on rule, which can be binary, text, or auto. 

 
Remounting a Filesystem 

$ mount -n -o remount/.

This command is re mount the root in read-write mode (we need the option -n because the mount command can’t write to the system
mount database when the root is read-only).

The /etc/fstab Filesystem Table 

To mount filesystems at boot time and take the drudgery out of the mount command, Linux systems keep a permanent
list of filesystems and options in /etc/fstab.

List of filesystems and options in /etc/fstab

proc /proc proc nodev,noexec,nosuid 0 0

UUID=70ccd6e7-6ae6-44f6-812c-51aab8036d29 / ext4 errors=remount-ro 0 1

UUID=592dcfd1-58da-4769-9ea8-5f412a896980 none swap sw 0 0 /dev/sr0 /cdrom iso9660 ro,user,nosuid,noauto 0 0

Each line corresponds to one filesystem, each of which is broken into six fields. 

These fields are as follows, in order from left to right:

The device or UUID   Most current Linux systems no longer use the device in /etc/fstab, preferring the UUID.

The mount points.      Indicates where to attach the filesystem.

The filesystem types.   You may not recognize swap in this list; this is a swap partition.

Options.                         Use long options separated by commas.

Backup information for use by the dump command. we should always use a 0 in this field.

The filesystem integrity test order. To ensure that fsck always runs on the root first, always set this to 1 for the root filesystem
and 2 for any other filesystems on a hard disk.

we can also try to mount all entries at once in /etc/fstab that do not contain the noauto option with this command:

$ mount -a

some new options, namely errors, noauto, and user, because they don’t apply outside the /etc/fstab file. In addition, you’ll often see
the defaults option here. 

The meanings of these options are

as follows:

defaults This uses the mount defaults: read-write mode, enable device files, executables, the setuid bit, and so on. 

Errors This ext2-specific parameter

sets the kernel behavior when the system has trouble mounting a filesystem. 

noauto This option tells a mount -a command to ignore the entry.

user This option allows unprivileged users to run mount on a particular entry, which can be handyfor allowing access to CD-ROM drives.

Because users can put a setuid-root file on removable media with another system.

 

Filesystem Capacity

We use $ df to view the size of the mounted filesystem.

Output will be.

Filesystem               1K-blocks                    Used      Available         Use% Mounted on 

host9p                     268435456                       0        268435456                    0% /mnt 

the description of the fields in the df output:

Filesystem. The filesystem device

1K-blocks. The total capacity of the filesystem in blocks of 1K

Used. The number of occupied blocks

Available. The number of free blocks

Use%. The percentage of blocks in use

Mounted on. The mount points

 

Checking and Repairing Filesystems

$ fsck. This command use for check and repair Linux filesystems (ext2,ext3,ext4), and is depending on when was the last time
filesystem was checked, the system runs the (fsck) during boot time to check whether the filesystem is in consistent state. 

The command is $ fsck /dev/sdb1.

 

Checking ext3 and ext4 Filesystems

ext3 and ext4 filesystem do not need to check manually because the journal ensures data integrity.

In ext2 we will mount a broken ex3, ext4 filesystem because the kernel will not mount an ext3 or ext4 with a nonempty journal.

To flush the journal in ext3 or ext4 filesystem to the regular filesystem database, run (e2fsck). 

Command $ e2fsck -fy /dev/disk-device.

 
Special-Purpose Filesystems

The special filesystem types in common use on Linux include the following:

proc  Mounted on /proc.

sysfs Mounted on /sys.

tmpfs Mounted on /run and other location.

 

**swap space**

swapping: it is the Linux virtual memory system can automatically move pieces of memory to and from a disk storage.

The disk area used to store memory page called swap space.


Using a Disk Partition as Swap Space

To use an entire disk partition as swap, we have to follow these three steps:

1.Make sure the partition is empty.

2.Run mkswap dev, where dev is the partition’s device. This command puts a swap signature on the partition.


We must keep in mind that many systems now use UUIDs instead of raw device names.

 

**Using a File as Swap Space**

we can create an empty file, initialize it as swap, and add it to the swap pool. 

Use these commands to create an empty file, initialize it as swap, and add it to the swap pool:

$ dd if=/dev/zero of=swap_file bs=1024k count=num_mb

$ mkswap swap_file

$ swapon swap_file

swap_file is the name of new swap file.

Inside a Traditional Filesystem

$ mkdir dir_1

$ mkdir dir_2

$ echo a > dir_1/file_1

$ echo b > dir_1/file_2

$ echo c > dir_1/file_3

$ echo d > dir_2/file_4

$ ln dir_1/file_3 dir_2/file_5

 

**Viewing Inode Details**

To view the inode numbers for any directory, we use this command ls -i . 

Here’s what we get at the root of this example. 

$ ls -i

 12 dir_1 7633 dir_2

 

                                                        **Chapter 5**


In this chapter, we will learn how the kernel starts— or boots.  A simplified view of the boot process looks like this:

1.    The machine’s BIOS or boot firmware loads and runs a boot loader.

2.    The boot loader finds the kernel image on disk, loads it into memory, and starts it.

3.    The kernel initializes the devices and its drivers.

4.    The kernel mounts the root filesystem.

5.    The kernel starts a program called init with a process ID of 1. This point is the user space start.

6.    init sets the rest of the system processes in motion.

7.    At some point, init starts a process allowing you to log in, usually at the end or near the end of the boot.

In this chapter is covers the first four stages.

 

**Startup Messages**

There are two ways to view the kernel’s boot and runtime diagnostic messages.

We often find this in /var/log/ kern.log, but depending on how system is configured, it might also be lumped together with a lot of
other system logs in /var/log/messages or elsewhere.

Use the dmesg command, but be sure to pipe the output to less because there will be much more than a screen’s worth. 

This is a sample of dmesg command:

$ dmesg

[    0.000000] Initializing cgroup subsys cpu

[    0.000000] Linux version 3.2.0-67-generic-pae (buildd@toyol) (gcc version 4.  6.3 (Ubuntu/Linaro 4.6.3-1ub  untu5) ) #101-Ubuntu 
SMP Tue Jul 15 18:04:54 UTC 2014 (Ubuntu 3.2.0-67.101-generic-pae 3.2.60)

[    0.000000] KERNEL supported cpus:

--snip--

[    2.986148] sr0: scsi3-mmc drive: 24x/8x writer dvd-ram cd/rw xa/form2 cdda tray

[    2.986153] cdrom: Uniform CD-ROM driver Revision: 3.20

[    2.986316] sr 1:0:0:0: Attached scsi CD-ROM sr0

[    2.986416] sr 1:0:0:0: Attached scsi generic sg1 type 5

[    3.007862] sda: sda1 sda2 < sda5 >

[    3.008658] sd 0:0:0:0: [sda] Attached SCSI disk

--snip--

After the kernel has started, the user-space startup procedure often generates messages. 

Kernel Initialization and Boot Options.

The Linux kernel initializes in this general order:

1.    CPU inspection

2.    Memory inspection

3.    Device bus discovery

4.    Device discovery

5.    Auxiliary kernel subsystem setup (networking, and so on)

6.    Root filesystem mount

7.    User space start

The first steps aren’t too remarkable, but when the kernel gets to devices, a question of dependencies arises. For example, the disk 
device drivers may depend on bus support and SCSI subsystem support.

Later in the initialization process, the kernel must mount a root file-system before starting init.

On some machines, we may need to load these kernel modules before the true root filesystem is mounted. 

 

**Kernel Parameters**

The parameters specify many different types of behavior, such as the amount of diagnostic output the kernel should produce and device
driver–specific options.

view the kernel parameters from the system’s boot by looking at the /proc/cmdline file:

$ cat /proc/cmdline. The output will be

BOOT_IMAGE=/boot/vmlinuz-3.2.0-67-generic-pae root=UUID=70ccd6e7-6ae6-44f6 

812c-51aab8036d29 ro quiet splash vt.handoff=7

The parameters are either simple one-word flags, such as ro , or key=value pairs, such as vt.handoff=7.

Most of the parameters are unimportant, such as the splash flag for displaying a splash screen, but one that is critical is
the root parameter. 

The root filesystem can be specified as a device file, such as in this example:

$ root=/dev/sda1

Most modern desktop systems, a UUID is more common.

$ root=UUID=70ccd6e7-6ae6-44f6-812c-51aab8036d29

The ro parameter is normal; it instructs the kernel to mount the root filesystem in read-only mode upon user space start. fsck will
check the root filesystem, after boot process remount the root filesystem in read-write mode.

                                                                
                                                                **Note**

A parameter that it does not understand, the Linux kernel will save the parameter, kernel later passes the parameter to init when 
performing the user space start.

For example, if we add -s to the kernel parameters, the kernel passes the -s to the init program to indicate that it should
start in single-user mode.

 

**Boot Loaders**

A boot loader starts the kernel, before the kernel and init start. The boot loader loads the kernel into memory, and then starts the 
kernel with a set of kernel parameters. 

Where is the kernel? kernel and its parameters are usually somewhere on the root filesystem, and should be easy to find.

**Boot Loader Tasks**

A Linux boot loader’s core functionality includes the ability to do the following:

Select among multiple kernels.

Switch between sets of kernel parameters.

Allow the user to manually override and edit kernel image names and parameters (for example, to enter single-user mode).

Provide support for booting other operating systems.

Boot loaders have become considerably more advanced since the inception of the Linux kernel, with features such as history and menu 
systems, but the basic need has always been flexibility in kernel image and parameter selection.

**Boot Loader Overview**

These are the main boot loaders:

GRUB. A near-universal standard on Linux systems

LILO. One of the first Linux boot loaders. ELILO is a UEFI version

SYSLINUX. Can be configured to run from many different kinds of filesystems

LOADLIN. Boots a kernel from MS-DOS efilinux. A UEFI boot loader intended to serve as a model and reference for other UEFI boot loaders

coreboot (formerly LinuxBIOS). A high-performance replacement for the PC BIOS that can
include a kernel

Linux Kernel EFISTUB. A kernel plugin for loading the kernel directly from the EFI/UEFI System Partition (ESP) found on recent systems

This book deals exclusively with GRUB. 

 

**GRUB Introduction**

GRUB stands for Grand Unified Boot Loader.

One of GRUB’s most important capabilities is filesystem navigation that allows for much easier kernel image and configuration 
selection. 

The interface is easy to navigate, but there’s a good chance that we may never seen it. 

To access the GRUB menu, press and hold SHIFT when your BIOS or firmware startup screen first appears. 

Press ESC to temporarily disable the automatic boot timeout after the GRUB menu appears.

To explore the boot loader, we must follow these steps:

1.    Reboot or power on Linux system.

2.    Hold down SHIFT during the BIOS/Firmware self-test and/or splash screen to get the GRUB menu.

3.    Press e to view the boot loader configuration commands for the default boot option. 

This screen tells us that for this configuration, the root is set with a UUID, the kernel image is /boot/vmlinuz-3.2.0-31-generic-pae,
and the kernel parameters include ro, quiet, and splash. The initial RAM filesystem is /boot/initrd.img-3.2.0-31-generic-paeWhy
are there multiple references to root, and why are they different? Why is insmod here? Isn’t that a Linux kernel feature normally run
by udevd.

 

Exploring Devices and Partitions with the GRUB Command Line

Listing Devices

To find of how GRUB refers to the devices on the system, we access the GRUB command line by pressing C at the boot menu or 
configuration editor. we should get the GRUB prompt:

grub>

ls. With no arguments, the output is a list of devices known to GRUB:

grub> ls

$ (hd0) (hd0,msdos1) (hd0,msdos5)

In this case, there is one main disk device denoted by (hd0) and the partitions (hd0,msdos1) and (hd0,msdos5). The msdos prefix
on the partitions tells that the disk contains an MBR partition table; it would begin with gpt for GPT. 

To get more detailed information, use ls -l. This command can be particularly useful because it displays any UUIDs of the partitions 
on  the disk. For example:

grub> ls -l

$ Device hd0: Not a known filesystem - Total size 426743808 sectors

Partition hd0,msdos1: Filesystem type ext2 – Last modification time

2015-09-18 20:45:00 Friday, UUID 4898e145-b064-45bd-b7b4-7326b00273b7 -

Partition start at 2048 - Total size 424644608 sectors

Partition hd0,msdos5: Not a known filesystem - Partition start at

424648704 - Total size 2093056 sectors

This particular disk has a Linux ext2/3/4 filesystem on the first MBR partition and a Linux swap signature on partition 5, which is a 
fairly common configuration

 

**File Navigation**

GRUB’s filesystem navigation capabilities.

grub> echo $root

$ hd0,msdos1

We use GRUB’s ls command to list the files and directories in that root, you can append a forward slash to the end of the partition:

grub> ls (hd0,msdos1)/

it is hard to remember the actual root partition, so we use the root variable to save yourself some time:

grub> ls ($root)/

The output is a short list of file and directory names on that partition’s filesystem, such as etc/, bin/, and dev/.


To take a deeper look into the files and directories on a partition in a similar manner. For example, to inspect the /boot directory,
start with the following:

grub> ls ($root)/boot

                                                               **NOTE**

We use the up and down arrow keys to flip through GRUB command history and the left and right arrows to edit the current command line.

Thestandard readline keys (CTRL-N, CTRL-P, and so on) also work.

We can use set command to  view all currently set GRUB variables:

grub> set ?=0

color_highlight=black/white

color_normal=white/black

--snip--

prefix=(hd0,msdos1)/boot/grub

root=hd0,msdos1

One of the most important of these variables is $prefix, the filesystem and directory where GRUB expects to find its configuration and
auxiliary support. 

When we finished with the GRUB command-line interface, we enter the boot command to boot the current configuration or just press ESC to 
return to the GRUB menu.

 

**GRUB Configuration**

The GRUB configuration directory contains the central configuration file (grub.cfg) and numerous loadable modules with 
a .mod suffix. (As GRUB versions progress, these modules will move into subdirectories such as i386-pc.)
The directory is usually /boot/grub or /boot/grub2. We won’t modify grub.cfg directly; instead,
we’ll use the grub-mkconfig command (or grub2-mkconfig on Fedora).

**Reviewing Grub.cfg**

First, take a quick look at grub.cfg to see how GRUB initializes its menu and kernel options.

we’ll see that the grub.cfg file consists of GRUB commands, which usually begin with many initialization steps followed by a
series of menu entries for different kernel and boot configurations.

If your grub.cfg file contains numerous menuentry commands, most of them are probably wrapped up inside a submenu command for older 
versions of the kernel so that they don’t crowd the GRUB menu.

**Generating a New Configuration File**

If you want to make changes to your GRUB configuration, you won’t edit your grub.cfg file directly because
it’s automatically generated and the system occasionally overwrites it. 

**GRUB Installation**

Installing GRUB is more involved than configuring it. Fortunately, we won’t normally have to worry about installation because our 
distribution should handle it for us.

**Installing GRUB on Your System**

Installing the boot loader requires we have to determine the following:

The target GRUB directory as seen by our currently running system.

That’s usually /boot/grub, but it might be different if we’re installing GRUB on another disk for use on another system.

The current device of the GRUB target disk.

For UEFI booting, the current mount point of the UEFI boot partition.

We have to remember that GRUB is a modular system, but in order to load modules, it must read the filesystem that contains the GRUB 
directory. 

Fortunately, GRUB comes with a utility called grub-install (not to be confused with Ubuntu’s install-grub), which performs most of the 
work of installing the GRUB files and configuration for you. For example, if the current disk is at /dev/sda and we want to install 
GRUB  on that disk with our current /boot/grub directory, we use this command to install GRUB on the MBR:

$  grub-install /dev/sda

                                                             **WARNING**

Incorrectly installing GRUB may break the bootup sequence on our system, so we don’t take this command lightly.

 

Installing GRUB on an External Storage Device

To install GRUB on a storage device outside the current system, we must manually specify the GRUB directory on that device as our 
current system now sees it. 

For example, say that we have a target device of /dev/sdc and that device’s root/boot filesystem (for example, /dev/sdc1)
is mounted on /mnt of your current system. 

This implies that when you install GRUB, your current system will see the GRUB files in /mnt/boot/grub. When
running grub-install, tell it where those files should go as follows:

$ grub-install --boot-directory=/mnt/boot /dev/sdc

 

**Installing GRUB with UEFI**

UEFI installation is supposed to be easier, because all we need to do is copy the boot loader into place. 

The grub-install command runs this if it’s available, so in theory all we need to do to install on an UEFI partition is the following:

$ grub-install --efi-directory=efi_dir –-bootloader-id=name

Unfortunately, many problems can crop up when installing a UEFI boot loader. 

For example, if we’re installing to a disk that will eventually end up in another system, we have to figure out how to announce that 
boot loader to the new system’s firmware. 

We have the biggest, and most problems is UEFI secure boot.

 

**UEFI Secure Boot Problems**

One of the newest problems affecting Linux installations is the secure boot feature found on recent PCs. 

When active, this mechanism in UEFI requires boot loaders to be digitally signed by a trusted authority to run. 

Microsoft has required vendors shipping Windows 8 to use secure boot. 

The result is that if you try to install an unsigned boot loader (which is most current Linux distributions), it will not load.

 
**Chainloading Other Operating Systems**

UEFI makes it relatively easy to support loading other operating systems because we can install multiple boot loaders in the EFI 
partition. 

 
**UEFI Boot**

PC manufacturers and software companies realized that the traditional PC BIOS is severely limited, so they decided to develop a
replacement called Extensible Firmware Interface (EFI) EFI took a while to catch on for most PCs, but now it’s fairly common.

The current standard is Unified EFI (UEFI), which includes features such as a built-in shell and the ability to read partition tables 
and navigate filesystems. The GPT partitioning scheme is part of the UEFI standard.

 
**How GRUB Works**

How it does its work:

1.    The PC BIOS or firmware initializes the hardware and searches its boot-order storage devices for boot code

2.    Upon finding the boot code, the BIOS/firmware loads and executes it. This is where GRUB begins.

3.    The GRUB core loads.

4.    The core initializes. At this point, GRUB can now access disks and filesystems.

5.    GRUB identifies its boot partition and loads a configuration there.

6.    GRUB gives the user a chance to change the configuration.

7.    During executing the configuration, GRUB may load additional code (modules) in the boot partition.

8.    GRUB executes a boot command to load and execute the kernel as specified by the configuration’s Linux    command.

Steps 3 and 4 of the preceding sequence, where the GRUB core loads, can be complicated due to the repeated inadequacies of traditional 
PC boot mechanisms. The biggest question is “Where is the GRUB core?” There are three basic possibilities:

Partially stuffed between the MBR and the beginning of the first partition

In a regular partition

In a special boot partition: a GPT boot partition, EFI System Partition (ESP), or elsewhere


Ref

1] Ward, Brian. How Linux works: what every superuser should
know. 2ND ed. San Francisco: No Starch Press, 2015.



