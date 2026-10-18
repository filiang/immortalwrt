# SPDX-License-Identifier: Apache-2.0
#
# Copyright (C) 2025 Anya Lin <https://github.com/muink>

include $(TOPDIR)/rules.mk

PKG_NAME:=scart
PKG_UPSTREAM_NAME:=scart
PKG_UPSTREAM_VERSION:=0.1.4
PKG_UPSTREAM_GITHASH:=
PKG_VERSION:=$(PKG_UPSTREAM_VERSION)$(if $(PKG_UPSTREAM_GITHASH),~$(call version_abbrev,$(PKG_UPSTREAM_GITHASH)))
PKG_RELEASE:=2

PKG_SOURCE_SUBDIR:=$(PKG_UPSTREAM_NAME)-$(PKG_UPSTREAM_VERSION)
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_SOURCE_SUBDIR)

ifeq ($(PKG_UPSTREAM_GITHASH),)
PKG_SOURCE_URL:=https://codeload.github.com/fcshark-org/scart/tar.gz/refs/tags/v$(PKG_UPSTREAM_VERSION)?
PKG_HASH:=57bffbaaf9b9add9605fd0a9a60321d479f3aacbbc55d8f1534c9565cdba620a

PKG_SOURCE:=$(PKG_SOURCE_SUBDIR).tar.gz
else
PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/fcshark-org/scart.git
PKG_SOURCE_VERSION:=$(PKG_UPSTREAM_GITHASH)
PKG_MIRROR_HASH:=skip

PKG_SOURCE:=$(PKG_SOURCE_SUBDIR)-$(PKG_SOURCE_VERSION).tar.gz
endif

PKG_MAINTAINER:=Anya Lin <hukk1996@gmail.com>
PKG_LICENSE:=Apache-2.0
PKG_LICENSE_FILES:=LICENSE

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=utils
	CATEGORY:=Utilities
	TITLE:=A Suspicious CA revoke tool
	URL:=https://github.com/fcshark-org/openwrt-scart
	DEPENDS:=+bash +jq-full +openssl-util
	PKGARCH:=all
endef

define Build/Configure
endef

define Build/Compile
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/scart $(1)/usr/sbin/scart
	$(INSTALL_BIN) $(CURDIR)/uci-defaults $(1)/etc/uci-defaults/99_$(PKG_NAME)
endef

define Package/$(PKG_NAME)/conffiles
endef

define Package/$(PKG_NAME)/postinst
endef

define Package/$(PKG_NAME)/prerm
endef

$(eval $(call BuildPackage,scart))
