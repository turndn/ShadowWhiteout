# ShadowWhiteout capable Overlay File Systems

The Shadow Whiteout is a concept presented in "Nesting Overlay File Systems with ShadowWhiteout" (APSys 2025).
This repository contains the research prototype of a Shadow Whiteout capable overlayfs for Linux 6.7.11.

## Setup

To get started, you need to build, install and run Linux kernel 6.7.11 on your machine.
The following commands have been tested on Linux kernel 6.7.11 running on Ubuntu 22.04 LTS.

### Prerequisites

1. Ensure that you are running the correct kernel version.
1. Clone the source code from GitHub.

```
~$ mkdir tmp
~$ cd tmp

# Check the currently running kernel version.
~/tmp$ uname -r
6.7.11

# Clone the source code.
~/tmp$ git clone https://github.com/turndn/ShadowWhiteout.git
```

Build the Shadow Whiteout capable overlayfs.

```
~/tmp$ cd ShadowWhiteout/fs/overlayfs
~/tmp/ShadowWhiteout/fs/overlayfs$ ls
Kconfig   copy_up.c  export.c  inode.c  overlayfs.h  params.c  readdir.c  util.c
Makefile  dir.c      file.c    namei.c  ovl_entry.h  params.h  super.c    xattrs.c
~/tmp/ShadowWhiteout/fs/overlayfs$ make -j8
```

Install the built module.
```
$ make reload
sudo rmmod overlay && sudo insmod overlay.ko
```

## Usage of Shadow Whiteout capable overlayfs

To configure a nesting overlay filesystem mount and see how it works, follow these steps.
In this example, we will use `/dev/nvme0n1p2` with ext4.

```
~/tmp/ShadowWhiteout/fs/overlayfs$ cd ../../../
~/tmp$ mkdir test
~/tmp$ cd test/
~/tmp/test$ mkdir mnt
```

Mount ext4 if needed (replace /dev/nvme0n1p2 with your device)

```
~/tmp/test$ sudo mount /dev/nvme0n1p2 mnt/
```

Set up the overlayfs directories.

```
~/tmp/test$ cd mnt
~/tmp/test/mnt$ mkdir mnt lower upper work
~/tmp/test/mnt$ ls
lower  mnt  upper  work
~/tmp/test/mnt$ mkdir lower/lower
~/tmp/test/mnt$ touch lower/lower/file0
~/tmp/test/mnt$ touch lower/lower/file1
~/tmp/test/mnt$ ls lower/lower
file0  file1
```

Mount overlayfs on ext4

```
~/tmp/test/mnt$ sudo mount -t overlay overlay -o lowerdir=lower,upperdir=upper,workdir=work mnt
~/tmp/test/mnt$ df -h | grep test
/dev/nvme0n1p2                           49G   23M   47G   1% /home/ishiguro/tmp/test/mnt
overlay                                  49G   23M   47G   1% /home/ishiguro/tmp/test/mnt/mnt
~/tmp/test/mnt$ ls mnt
lower
```

Create nested overlayfs

```
~/tmp/test/mnt$ cd mnt/
~/tmp/test/mnt/mnt$ mkdir upper work mnt
~/tmp/test/mnt/mnt$ ls
lower  mnt  upper  work

~/tmp/test/mnt/mnt$ rm lower/file0
~/tmp/test/mnt/mnt$ ls lower
file1
```

Mount overlayfs on overlayfs.

```
~/tmp/test/mnt/mnt$ sudo mount -t overlay overlay -o lowerdir=lower,upperdir=upper,workdir=work mnt
~/tmp/test/mnt/mnt$ df -h | grep test
/dev/nvme0n1p2                           49G   23M   47G   1% /home/ishiguro/tmp/test/mnt
overlay                                  49G   23M   47G   1% /home/ishiguro/tmp/test/mnt/mnt
overlay                                  49G   23M   47G   1% /home/ishiguro/tmp/test/mnt/mnt/mnt
~/tmp/test/mnt/mnt$ sudo dmesg | tail -n 4
[ 4499.279623] overlayfs: [Modified] allow to use overlayfs as upperdir
[ 4499.279630] overlayfs: [Modified] allow to use overlayfs as upperdir
[ 4499.279802] overlayfs: upper fs does not support tmpfile.
[ 4499.279913] overlayfs: fs on 'lower' does not support file handles, falling back to xino=off.
~/tmp/test/mnt/mnt$ ls mnt
file1
~/tmp/test/mnt/mnt$ rm mnt/file1
~/tmp/test/mnt/mnt$ ls mnt
```

Check normal and Shadow Whiteout files

```
~/tmp/test/mnt/mnt$ cd ../../

# See the normal whiteout file
~/tmp/test$ ls -l mnt/upper/lower/file0
c--------- 2 root root 0, 0 Sep 30 15:25 mnt/upper/lower/file0
~/tmp/test$ sudo attr -l mnt/upper/lower/file0

# See the shadow whiteout file
~/tmp/test$ ls -l mnt/upper/upper/file1
c--------- 1 root root 0, 1 Sep 30 15:25 mnt/upper/upper/file1
~/tmp/test$ sudo `which attr` -l mnt/upper/upper/file1
Attribute "overlay.overlay.whiteout" has a 0 byte value for mnt/upper/upper/file1
```

## Branches

* `main`: overlayfs with Shadow Whiteout.
* `shadow-whiteout-regular`: overlayfs with Shadow Whiteout regular file version.

## Changes from the original

Please refer to `swhiteout.patch` for details on chages made.

## Links and citation

You can see the paper on

* https://doi.org/10.1145/3725783.3764408
* https://turndn.github.io/#swhiteout_apsys25

```
Kenta Ishiguro, Kohei Hayama, Ayase Yokoyama, Hiroshi Yamada, and Toshio Hirotsu.
Nesting Overlay File Systems with ShadowWhiteout.
In Proceedings of the 16th ACM SIGOPS Asia-Pacific Workshop on Systems (APSys '25), Oct. 2025 
```

