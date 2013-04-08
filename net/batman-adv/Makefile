#
# Copyright (C) 2010 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id: Makefile 5624 2006-11-23 00:29:07Z nbd $

include $(TOPDIR)/rules.mk

PKG_NAME:=batman-adv-devel

PKG_BATMAN_REV:=fe476985951d185caf16aa59e78088365c3b5dd0
PKG_BATCTL_REV:=59b559c4138d006beabf79e8f97767e99726dce7

PKG_BATMAN_BRANCH:=$(subst ",,$(CONFIG_KMOD_BATMAN_ADV_DEVEL_BRANCH_NAME))
PKG_BATMAN_SHORTREV:=$(shell echo $(PKG_BATMAN_REV) | cut -c1-7)
PKG_BATMAN_FNAME:=batman-adv-devel-$(PKG_BATMAN_BRANCH)-$(PKG_BATMAN_SHORTREV).tar.gz
PKG_BATMAN_DIR:=batman-adv-$(PKG_BATMAN_SHORTREV)
PKG_BATCTL_SHORTREV:=$(shell echo $(PKG_BATCTL_REV) | cut -c1-7)
PKG_BATCTL_FNAME:=batctl-devel-$(PKG_BATCTL_SHORTREV).tar.gz
PKG_BATCTL_DIR:=batctl-$(PKG_BATCTL_SHORTREV)

PKG_SOURCE_URL:=http://git.open-mesh.org/snapshot/
PKG_SOURCE:=$(PKG_BATMAN_FNAME)

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/$(PKG_BATMAN_DIR)
PKG_BATCTL_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/$(PKG_BATCTL_DIR)

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/kernel.mk

define KernelPackage/batman-adv-devel
  URL:=http://www.open-mesh.org/
  MAINTAINER:=Marek Lindner <lindner_marek@yahoo.de>
  SUBMENU:=Network Support
  DEPENDS:=+kmod-lib-crc16
  TITLE:=B.A.T.M.A.N. Adv devel
  FILES:=$(PKG_BUILD_DIR)/batman-adv.$(LINUX_KMOD_SUFFIX)
  AUTOLOAD:=$(call AutoLoad,50,batman-adv)
endef

define KernelPackage/batman-adv-devel/description
B.A.T.M.A.N. advanced is a kernel module which allows to
build layer 2 mesh networks. This package builds the latest
development version ($(PKG_BATMAN_SHORTREV)) of the kernel module plus 
its user space configuration & managerment tool batctl.
endef

define KernelPackage/batman-adv-devel/config
	source "$(SOURCE)/Config.in"
endef

MAKE_BATMAN_ADV_ARGS += \
	CROSS_COMPILE="$(TARGET_CROSS)" \
	KERNELPATH="$(LINUX_DIR)" \
	ARCH="$(LINUX_KARCH)" \
	PATH="$(TARGET_PATH)" \
	SUBDIRS="$(PKG_BUILD_DIR)" \
	PWD="$(PKG_BUILD_DIR)" \
	LINUX_VERSION="$(LINUX_VERSION)" \
	CONFIG_BATMAN_ADV_DEBUG=$(if $(CONFIG_KMOD_BATMAN_ADV_DEVEL_DEBUG_LOG),y,n) \
	CONFIG_BATMAN_ADV_BLA=$(if $(CONFIG_KMOD_BATMAN_ADV_DEVEL_BLA),y,n) \
	CONFIG_BATMAN_ADV_STATS=y \
	CONFIG_BATMAN_ADV_NC=y \
	REVISION="$(PKG_BATMAN_BRANCH)-$(PKG_BATMAN_SHORTREV)" all

MAKE_BATCTL_ARGS += \
	CFLAGS="$(TARGET_CFLAGS)" \
	CCFLAGS="$(TARGET_CFLAGS)" \
	OFLAGS="$(TARGET_CFLAGS)" \
	REVISION="$(PKG_BATCTL_SHORTREV)" \
	CC="$(TARGET_CC)" \
	NODEBUG=1 \
	UNAME="Linux" \
	DESTDIR="$(PKG_INSTALL_DIR)" \
	STRIP="/bin/true" \
	batctl install

ifneq ($(DEVELOPER)$(CONFIG_KMOD_BATMAN_ADV_DEVEL_BATCTL),)
define Download/batctl
  FILE:=$(PKG_BATCTL_FNAME)
  URL:=$(PKG_SOURCE_URL)
endef
$(eval $(call Download,batctl))

BATCTL_EXTRACT = tar xzf "$(DL_DIR)/$(PKG_BATCTL_FNAME)" -C "$(BUILD_DIR)/$(PKG_NAME)"
BATCTL_PATCH = $(call Build/DoPatch,"$(PKG_BATCTL_BUILD_DIR)","$(PATCH_DIR)","*batctl*")
BATCTL_BUILD = $(MAKE) -C $(PKG_BATCTL_BUILD_DIR) $(MAKE_BATCTL_ARGS)
BATCTL_INSTALL = $(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/local/sbin/batctl $(1)/usr/sbin/
endif

KPATCH ?= $(PATCH)
define Build/DoPatch
	@if [ -d "$(2)" ]; then \
		if [ "$$$$(ls $(2) | grep -Ec $(3))" -gt 0 ]; then \
			$(KPATCH) "$(1)" "$(2)" "$(3)"; \
		fi; \
	fi
endef

define Build/Patch
	$(call Build/DoPatch,"$(PKG_BUILD_DIR)","$(PATCH_DIR)","*batman*")
	$(BATCTL_EXTRACT)
	$(BATCTL_PATCH)
endef

define Build/Compile
	$(MAKE) -C "$(PKG_BUILD_DIR)" $(MAKE_BATMAN_ADV_ARGS)
	$(BATCTL_BUILD)
endef

define Build/Clean
        rm -rf $(BUILD_DIR)/$(PKG_NAME)/
endef

define KernelPackage/batman-adv-devel/install
	$(INSTALL_DIR) $(1)/etc/config $(1)/etc/hotplug.d/net $(1)/lib/batman-adv $(1)/usr/sbin
	$(INSTALL_DATA) ./files/etc/config/batman-adv $(1)/etc/config
	$(INSTALL_DATA) ./files/lib/batman-adv/config.sh $(1)/lib/batman-adv
	$(INSTALL_BIN) ./files/etc/hotplug.d/net/99-batman-adv $(1)/etc/hotplug.d/net
	$(INSTALL_BIN) ./files/lib/netifd/proto/batadv.sh $(1)/lib/netifd/proto
	$(BATCTL_INSTALL)
endef

$(eval $(call KernelPackage,batman-adv-devel))
