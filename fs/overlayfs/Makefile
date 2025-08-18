# SPDX-License-Identifier: GPL-2.0-only
#
# Makefile for the overlay filesystem.
#
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

obj-$(CONFIG_OVERLAY_FS) += overlay.o

overlay-objs := super.o namei.o util.o inode.o file.o dir.o readdir.o \
		copy_up.o export.o params.o xattrs.o

all:
	$(MAKE) -C $(KDIR) M=$(PWD) overlay.ko

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean

reload:
	sudo rmmod overlay && sudo insmod overlay.ko
