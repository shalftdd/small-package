# SPDX-License-Identifier: GPL-3.0-only
#
# Copyright (C) 2021 ImmortalWrt.org

include $(TOPDIR)/rules.mk

PKG_NAME:=v2rayA
PKG_VERSION:=1.5.6.2
PKG_RELEASE:=2

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/v2rayA/v2rayA/tar.gz/v$(PKG_VERSION)?
PKG_HASH:=skip
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)/service

PKG_LICENSE:=AGPL-3.0-only
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Tianling Shen <cnsztl@immortalwrt.org>

PKG_CONFIG_DEPENDS:= \
	CONFIG_V2RAYA_COMPRESS_GOPROXY \
	CONFIG_V2RAYA_COMPRESS_UPX \

PKG_BUILD_DEPENDS:=golang/host node/host node-yarn/host
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0

GO_PKG:=github.com/v2rayA/v2rayA
GO_PKG_LDFLAGS:=-s -w
GO_PKG_LDFLAGS_X:=$(GO_PKG)/conf.Version=$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

TAR_OPTIONS+= --strip-components 1
TAR_CMD=$(HOST_TAR) -C $(1)/.. $(TAR_OPTIONS)

define Package/v2raya
  TITLE:=A Linux web GUI client of Project V
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  DEPENDS:=$(GO_ARCH_DEPENDS) \
    +ca-bundle \
    +iptables-mod-conntrack-extra \
    +iptables-mod-extra \
    +iptables-mod-filter \
    +iptables-mod-tproxy \
    +kmod-ipt-nat6 \
    +xray-core
  URL:=https://v2raya.org
endef

define Package/v2raya/description
  v2rayA is a V2Ray Linux client supporting global transparent proxy,
  compatible with SS, SSR, Trojan(trojan-go), PingTunnel protocols.
endef

define Package/v2raya/config
config V2RAYA_COMPRESS_GOPROXY
	bool "Compiling with GOPROXY proxy"
	default n

config V2RAYA_COMPRESS_UPX
	bool "Compress executable files with UPX"
	depends on !mips64
	default n
endef

ifneq ($(CONFIG_V2RAYA_COMPRESS_GOPROXY),)
	export GO111MODULE=on
	export GOPROXY=https://goproxy.bj.bcebos.com
endif

define Package/v2raya/conffiles
/etc/v2raya/
/etc/config/v2raya
endef

define Build/Compile
	( \
		pushd $(PKG_BUILD_DIR)/../gui ; \
		yarn ; \
		OUTPUT_DIR=../service/server/router/web yarn build ; \
		popd ; \
		$(call GoPackage/Build/Compile) ; \
	)
ifneq ($(CONFIG_V2RAYA_COMPRESS_UPX),)
	$(STAGING_DIR_HOST)/bin/upx --lzma --best $(GO_PKG_BUILD_BIN_DIR)/v2rayA
endif
endef

define Package/v2raya/install
	$(call GoPackage/Package/Install/Bin,$(PKG_INSTALL_DIR))
	$(INSTALL_DIR) $(1)/usr/bin/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/v2rayA $(1)/usr/bin/v2raya

	$(INSTALL_DIR) $(1)/etc/config/
	$(INSTALL_CONF) $(CURDIR)/files/v2raya.config $(1)/etc/config/v2raya
	$(INSTALL_DIR) $(1)/etc/init.d/
	$(INSTALL_BIN) $(CURDIR)/files/v2raya.init $(1)/etc/init.d/v2raya
endef

$(eval $(call GoBinPackage,v2raya))
$(eval $(call BuildPackage,v2raya))
