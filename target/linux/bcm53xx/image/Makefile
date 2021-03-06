#
# Copyright (C) 2013 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

define Image/Prepare
	rm -f $(KDIR)/fs_mark
	echo -ne '\xde\xad\xc0\xde' > $(KDIR)/fs_mark
	$(call prepare_generic_squashfs,$(KDIR)/fs_mark)

	# For UBI we want only one extra block
	rm -f $(KDIR)/ubi_mark
	echo -ne '\xde\xad\xc0\xde' > $(KDIR)/ubi_mark
endef

define Build/lzma-d16
	$(STAGING_DIR_HOST)/bin/lzma e $@ -d16 $(1) $@.new
	mv $@.new $@
endef

define Build/trx-serial
	$(STAGING_DIR_HOST)/bin/trx \
		-o $@.new \
		-m 33554432 \
		-f $(IMAGE_KERNEL) -a 1024 \
		-f $@ -a 0x10000 -A $(KDIR)/fs_mark
	mv $@.new $@
endef

define Build/trx-nand
	# kernel: always use 4 MiB (-28 B or TRX header) to allow upgrades even
	#	  if it grows up between releases
	# root: UBI with one extra block containing UBI mark to trigger erasing
	#	rest of partition
	$(STAGING_DIR_HOST)/bin/trx \
		-o $@.new \
		-m 33554432 \
		-f $(IMAGE_KERNEL) -a 0x20000 -b 0x400000 \
		-f $@ \
		-A $(KDIR)/ubi_mark -a 0x20000
	mv $@.new $@
endef

define Build/asus-trx
	$(STAGING_DIR_HOST)/bin/asustrx \
		-p $(PRODUCTID) -i $@ -o $@.new
	mv $@.new $@
endef

define Build/seama-nand
	# Seama entity
	$(STAGING_DIR_HOST)/bin/oseama \
		entity $@.entity \
		-m "dev=/dev/mtdblock/7" \
		-m "type=firmware" \
		-f $(IMAGE_KERNEL) \
		-b 0x400000 \
		-f $@ \
		-f $(KDIR)/ubi_mark
	# Seama container
	$(STAGING_DIR_HOST)/bin/seama \
		-s $@ \
		-m "signature=$(SIGNATURE)" \
		-i $@.entity
endef

DEVICE_VARS += PRODUCTID SIGNATURE NETGEAR_BOARD_ID NETGEAR_REGION

USB2_PACKAGES := kmod-usb-ohci kmod-usb2
USB3_PACKAGES := kmod-usb-ohci kmod-usb2 kmod-usb3

define Device/Default
  # .dtb files are prefixed by SoC type, e.g. bcm4708- which is not included in device/image names
  # extract the full dtb name based on the device info
  DEVICE_DTS := $(patsubst %.dtb,%,$(notdir $(wildcard $(if $(IB),$(KDIR),$(DTS_DIR))/*-$(1).dtb)))
  KERNEL := kernel-bin | append-dtb | lzma-d16
  KERNEL_DEPENDS = $$(wildcard $(DTS_DIR)/$$(DEVICE_DTS).dts)
  FILESYSTEMS := squashfs
  KERNEL_NAME := zImage
  IMAGE_NAME = $$(IMAGE_PREFIX)-$$(1).$$(2)
  IMAGES := trx
  BLOCKSIZE := 128KiB
  PAGESIZE := 2048
  UBINIZE_OPTS :=
  IMAGE/trx := append-ubi | trx-nand
endef

define Device/asus
  IMAGES := trx
  IMAGE/trx := append-ubi | trx-nand | asus-trx
endef

define Device/asus-rt-ac56u
  DEVICE_TITLE := Asus RT-AC56U
  DEVICE_PACKAGES := kmod-b43 $(USB3_PACKAGES)
endef
TARGET_DEVICES += asus-rt-ac56u

define Device/asus-rt-ac68u
  DEVICE_TITLE := Asus RT-AC68U
  DEVICE_PACKAGES := $(USB3_PACKAGES)
endef
TARGET_DEVICES += asus-rt-ac68u

define Device/asus-rt-ac87u
  DEVICE_TITLE := Asus RT-AC87U
  DEVICE_PACKAGES := $(USB3_PACKAGES)
endef
TARGET_DEVICES += asus-rt-ac87u

define Device/asus-rt-n18u
  DEVICE_TITLE := Asus RT-N18U
  DEVICE_PACKAGES := $(USB3_PACKAGES)
endef
TARGET_DEVICES += asus-rt-n18u

define Device/buffalo-wxr-1900dhp
  DEVICE_TITLE := Buffalo WXR-1900DHP
  DEVICE_PACKAGES := $(USB3_PACKAGES)
endef
TARGET_DEVICES += buffalo-wxr-1900dhp

define Device/buffalo-wzr-600dhp2
  DEVICE_TITLE := Buffalo WZR-600DHP2
  DEVICE_PACKAGES := kmod-b43 $(USB2_PACKAGES)
endef
TARGET_DEVICES += buffalo-wzr-600dhp2

define Device/buffalo-wzr-900dhp
  DEVICE_TITLE := Buffalo WZR-900DHP
  DEVICE_PACKAGES := kmod-b43 $(USB3_PACKAGES)
endef
TARGET_DEVICES += buffalo-wzr-900dhp

define Device/buffalo-wzr-1750dhp
  DEVICE_TITLE := Buffalo WZR-1750DHP
  DEVICE_PACKAGES := kmod-b43 $(USB3_PACKAGES)
endef
TARGET_DEVICES += buffalo-wzr-1750dhp

define Device/dlink
  IMAGES := bin
  IMAGE/bin := append-ubi | seama-nand
endef

define Device/dlink-dir-885l
  DEVICE_TITLE := D-Link DIR-885L
  DEVICE_PACKAGES := kmod-brcmfmac $(USB3_PACKAGES)
  $(Device/dlink)
  SIGNATURE := wrgac42_dlink.2015_dir885l
endef
TARGET_DEVICES += dlink-dir-885l

define Device/linksys-ea6300-v1
  DEVICE_TITLE := Linksys EA6300 V1
  DEVICE_PACKAGES := kmod-b43 $(USB3_PACKAGES)
endef
TARGET_DEVICES += linksys-ea6300-v1

define Device/netgear
  IMAGES := chk
  IMAGE/chk := append-ubi | trx-nand | netgear-chk
  NETGEAR_REGION := 1
endef

define Device/netgear-r6250
  DEVICE_TITLE := Netgear R6250
  DEVICE_PACKAGES := kmod-b43 $(USB3_PACKAGES)
  $(Device/netgear)
  NETGEAR_BOARD_ID := U12H245T00_NETGEAR
endef

define Device/netgear-r6300-v2
  DEVICE_TITLE := Netgear R6300 V2
  DEVICE_PACKAGES := kmod-b43 $(USB3_PACKAGES)
  $(Device/netgear)
  NETGEAR_BOARD_ID := U12H240T00_NETGEAR
endef

define Device/netgear-r7000
  DEVICE_TITLE := Netgear R7000
  DEVICE_PACKAGES := $(USB3_PACKAGES)
  $(Device/netgear)
  NETGEAR_BOARD_ID := U12H270T00_NETGEAR
endef
TARGET_DEVICES += netgear-r7000

define Device/netgear-r7900
  DEVICE_TITLE := Netgear R7900
  DEVICE_PACKAGES := kmod-brcmfmac $(USB3_PACKAGES)
  $(Device/netgear)
  NETGEAR_BOARD_ID := U12H315T30_NETGEAR
endef
TARGET_DEVICES += netgear-r7900

define Device/netgear-r8000
  DEVICE_TITLE := Netgear R8000
  DEVICE_PACKAGES := kmod-brcmfmac $(USB3_PACKAGES)
  $(Device/netgear)
  NETGEAR_BOARD_ID := U12H315T00_NETGEAR
endef

define Device/netgear-r8500
  DEVICE_TITLE := Netgear R8500
  DEVICE_PACKAGES := kmod-brcmfmac $(USB3_PACKAGES)
  $(Device/netgear)
  NETGEAR_BOARD_ID := U12H334T00_NETGEAR
endef
# TARGET_DEVICES += netgear-r8500

define Device/smartrg-sr400ac
  DEVICE_TITLE := SmartRG SR400ac
  DEVICE_PACKAGES := kmod-brcmfmac $(USB3_PACKAGES)
  IMAGES := trx
  IMAGE/trx := append-rootfs | trx-serial
endef

define Device/tenda-ac9
  DEVICE_TITLE := Tenda AC9
  DEVICE_PACKAGES := $(USB2_PACKAGES)
  IMAGES := trx
  IMAGE/trx := append-rootfs | trx-serial
endef
# TARGET_DEVICES += tenda-ac9

TARGET_DEVICES += \
	netgear-r6250 netgear-r6300-v2 netgear-r8000 \
	smartrg-sr400ac

$(eval $(call BuildImage))
