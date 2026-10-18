# SPDX-License-Identifier: MIT
#
# Copyright (C) 2025-2026 Anya Lin <hukk1996@gmail.com>

include $(TOPDIR)/rules.mk

PKG_NAME:=iusebash
PKG_VERSION:=0.2026.06.15
PKG_RELEASE:=1

PKG_MAINTAINER:=Anya Lin <hukk1996@gmail.com>
PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=utils
	CATEGORY:=Utilities
	TITLE:=Use bash as the default terminal
	URL:=https://github.com/muink/openwrt-iusebash
	DEPENDS:=+bash
	PKGARCH:=all
endef

Build/Compile=

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) $(CURDIR)/files/iusebash.init $(1)/etc/init.d/iusebash

	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) $(CURDIR)/files/iusebash.defaults $(1)/etc/uci-defaults/99_iusebash
endef

define Package/$(PKG_NAME)/prerm
#!/bin/sh
sed -i 's|/bin/bash|/bin/ash|g' "$$IPKG_INSTROOT/etc/passwd"
if [ -z "$$IPKG_INSTROOT" ]; then
	# system
	rm -f /tmp/.busybox_ash_history 2>/dev/null
fi
exit 0
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
