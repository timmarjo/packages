#
# Copyright (C) 2012-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=nginx
PKG_VERSION:=1.25.0
PKG_RELEASE:=1

PKG_SOURCE:=nginx-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://nginx.org/download/
PKG_HASH:=5ed44d45943272a4e8a5bcf4434237210f2de31b903fca5e381c1bbd7eee1e8c

PKG_MAINTAINER:=Thomas Heil <heil@terminal-consulting.de> \
				Ansuel Smith <ansuelsmth@gmail.com>
PKG_LICENSE:=2-clause BSD-like license
PKG_CPE_ID:=cpe:/a:nginx:nginx

PKG_FIXUP:=autoreconf
PKG_BUILD_PARALLEL:=1
PKG_INSTALL:=1
PKG_BUILD_FLAGS:=gc-sections

# 3rd-party modules
PKG_MOD_EXTRA := \
	lua \
	rtmp \
	dav-ext \
	naxsi \
	brotli \
	headers-more \
	ts \
	ubus
# official and 3rd-party modules
PKG_MOD_ALL := \
	$(PKG_MOD_EXTRA) \
	stream
PKG_MOD_PATCHED := $(shell find patches -mindepth 1 -maxdepth 1 -type d -name 'nginx-mod-*' | sed -E 's,^patches/nginx-mod-,,')

PKG_CONFIG_DEPENDS := \
	CONFIG_NGINX_DAV \
	CONFIG_NGINX_FLV \
	CONFIG_NGINX_STUB_STATUS \
	CONFIG_NGINX_HTTP_CHARSET \
	CONFIG_NGINX_HTTP_GZIP \
	CONFIG_NGINX_HTTP_GZIP_STATIC \
	CONFIG_NGINX_HTTP_SSI \
	CONFIG_NGINX_HTTP_USERID \
	CONFIG_NGINX_HTTP_ACCESS \
	CONFIG_NGINX_HTTP_AUTH_BASIC \
	CONFIG_NGINX_HTTP_AUTH_REQUEST \
	CONFIG_NGINX_HTTP_AUTOINDEX \
	CONFIG_NGINX_HTTP_GEO \
	CONFIG_NGINX_HTTP_MAP \
	CONFIG_NGINX_HTTP_SPLIT_CLIENTS \
	CONFIG_NGINX_HTTP_REFERER \
	CONFIG_NGINX_HTTP_REWRITE \
	CONFIG_NGINX_HTTP_PROXY \
	CONFIG_NGINX_HTTP_FASTCGI \
	CONFIG_NGINX_HTTP_UWSGI \
	CONFIG_NGINX_HTTP_SCGI \
	CONFIG_NGINX_HTTP_MEMCACHED \
	CONFIG_NGINX_HTTP_LIMIT_CONN \
	CONFIG_NGINX_HTTP_LIMIT_REQ \
	CONFIG_NGINX_HTTP_EMPTY_GIF \
	CONFIG_NGINX_HTTP_BROWSER \
	CONFIG_NGINX_HTTP_UPSTREAM_HASH \
	CONFIG_NGINX_HTTP_UPSTREAM_IP_HASH \
	CONFIG_NGINX_HTTP_UPSTREAM_LEAST_CONN \
	CONFIG_NGINX_HTTP_UPSTREAM_KEEPALIVE \
	CONFIG_NGINX_HTTP_UPSTREAM_ZONE \
	CONFIG_NGINX_HTTP_CACHE \
	CONFIG_NGINX_HTTP_V2 \
	CONFIG_NGINX_PCRE \
	CONFIG_NGINX_HTTP_REAL_IP \
	CONFIG_NGINX_HTTP_SECURE_LINK \
	CONFIG_OPENSSL_ENGINE \
	CONFIG_OPENSSL_WITH_NPN \
	$(foreach m,$(PKG_MOD_EXTRA),CONFIG_PACKAGE_$(m))

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/nls.mk

define Package/nginx/default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=Nginx web server
  URL:=http://nginx.org/
  DEPENDS:=+libopenssl +libpthread
  # TODO: add PROVIDES when removing nginx
  # PROVIDES:=nginx
endef

define Package/nginx/description
 nginx is an HTTP and reverse proxy server, as well as a mail proxy server, \
 written by Igor Sysoev.
endef

define Package/nginx/conffiles
/etc/nginx/
endef

define Package/nginx-ssl
  $(Package/nginx/default)
  TITLE += with SSL support
  VARIANT:=ssl
  DEPENDS+= +NGINX_PCRE:libpcre \
	+NGINX_PCRE:nginx-ssl-util +!NGINX_PCRE:nginx-ssl-util-nopcre \
	+NGINX_HTTP_GZIP:zlib +NGINX_DAV:libxml2
  EXTRA_DEPENDS:=nginx-ssl-util$(if $(CONFIG_NGINX_PCRE),,-nopcre) (>=1.5-1) (<2)
  CONFLICTS:=nginx-full
endef

Package/nginx-ssl/description = $(Package/nginx/description) \
  This variant is compiled with SSL support enabled. To enable additional module \
  select them in the nginx default configuration menu.

define Package/nginx-ssl/config
  source "$(SOURCE)/Config_ssl.in"
endef

Package/nginx-ssl/conffiles = $(Package/nginx/conffiles)

IsEnabled = $(or $(filter full,$(BUILD_VARIANT)),$(CONFIG_$(1)))
IsDisabled = $(if $(call IsEnabled,$(1)),,1)

config_files := mime.types \
	$(if $(call IsEnabled,NGINX_HTTP_CHARSET),koi-utf koi-win win-utf) \
	$(if $(call IsEnabled,NGINX_HTTP_FASTCGI),fastcgi_params) \
	$(if $(call IsEnabled,NGINX_HTTP_UWSGI),uwsgi_params) \
	$(if $(call IsEnabled,NGINX_HTTP_SCGI),scgi_params)

define Package/nginx-ssl/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/sbin/nginx $(1)/usr/sbin/
	$(INSTALL_DIR) $(1)/etc/nginx/conf.d
	$(INSTALL_DATA) $(addprefix $(PKG_INSTALL_DIR)/etc/nginx/,$(config_files)) $(1)/etc/nginx/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/nginx.init $(1)/etc/init.d/nginx
endef

define Package/nginx-ssl/prerm
#!/bin/sh
[ -z "$${IPKG_INSTROOT}" ] || exit 0
[ "$${PKG_UPGRADE}" = "1" ] && exit 0
eval $$(/usr/bin/nginx-util get_env)
[ "$$(uci get "nginx.$${LAN_NAME}.$${MANAGE_SSL}")" = "self-signed" ] || exit 0
rm -f "$$(uci get "nginx.$${LAN_NAME}.ssl_certificate")"
rm -f "$$(uci get "nginx.$${LAN_NAME}.ssl_certificate_key")"
exit 0
endef

define Package/nginx-full
  $(Package/nginx/default)
  TITLE += with ALL module selected
  DEPENDS+=+libpcre +nginx-ssl-util +zlib +libxml2 \
    $(foreach m,$(PKG_MOD_ALL),+nginx-mod-$(m))
  EXTRA_DEPENDS:=nginx-ssl-util (>=1.5-1) (<2)
  VARIANT:=full
  PROVIDES += nginx-ssl
endef

Package/nginx-full/description = $(Package/nginx/description) \
  This variant is compiled with ALL module selected.

Package/nginx-full/install = $(Package/nginx-ssl/install)

Package/nginx-full/prerm = $(Package/nginx-ssl/prerm)

Package/nginx-full/conffiles = $(Package/nginx/conffiles)

define Package/nginx-mod-luci
  TITLE:=Nginx on LuCI
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=Support file for Nginx
  URL:=http://nginx.org/
  DEPENDS:=+uwsgi +uwsgi-luci-support +nginx-ssl +nginx-mod-ubus
  # TODO: add PROVIDES when removing nginx-mod-luci-ssl
  # PROVIDES:=nginx-mod-luci-ssl
endef

define Package/nginx-mod-luci/description
 Support file for LuCI in nginx. Include custom nginx configuration, autostart script for uwsgi.
endef

define Package/nginx-mod-luci/install
	$(INSTALL_DIR) $(1)/etc/nginx/conf.d
	$(INSTALL_CONF) ./files-luci-support/luci.locations $(1)/etc/nginx/conf.d/
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) ./files-luci-support/60_nginx-luci-support $(1)/etc/uci-defaults/60_nginx-luci-support
endef

define Download/nginx-mod-headers-more
  VERSION:=bea1be3bbf6af28f6aa8cf0c01c07ee1637e2bd0
  URL:=https://github.com/openresty/headers-more-nginx-module.git
  MIRROR_HASH:=3617bbf7a935208a1d8d5f86a8f9b770f6987e4d2b5663a9ab1b777217e3066b
  PROTO:=git
endef


define Download/nginx-mod-brotli
  VERSION:=e505dce68acc190cc5a1e780a3b0275e39f160ca
  URL:=https://github.com/google/ngx_brotli.git
  MIRROR_HASH:=04847f11ef808fed50f44b2af0ef3abf59ff0ffc06dfc7394d9ab51d53fef31f
  PROTO:=git
endef

define Download/nginx-mod-rtmp
  VERSION:=f0ea62342a4eca504b311cd5df910d026c3ea4cf
  URL:=https://github.com/ut0mt8/nginx-rtmp-module.git
  MIRROR_HASH:=d3f58066f0f858ed79f7f2b0c9b89de2ccc512c94ab3d0625f6dcff3df0b72c1
  PROTO:=git
endef

define Download/nginx-mod-ts
  VERSION:=ef2f874d95cc75747eb625a292524a702aefb0fd
  URL:=https://github.com/arut/nginx-ts-module.git
  MIRROR_HASH:=73938950bb286d40d9e54b0994d1a63827340c1156c72eb04d7041b25b20ec18
  PROTO:=git
endef

define Download/nginx-mod-naxsi
  VERSION:=951123ad456bdf5ac94e8d8819342fe3d49bc002
  URL:=https://github.com/nbs-system/naxsi.git
  MIRROR_HASH:=c734cae19a596affadd62a2df1b58d3df8d1364093a4e80a7cd1ab4555963535
  PROTO:=git
endef

define Download/nginx-mod-lua
  VERSION:=68acad14e4a8f42e31d4a4bb5ed44d6f5b55fc1c
  URL:=https://github.com/openresty/lua-nginx-module.git
  MIRROR_HASH:=366f24e1ba6221e34f6ba20ab29146438438f88c89fd71f9500d169b3f5aedf0
  PROTO:=git
endef

define Download/nginx-mod-dav-ext
  VERSION:=f5e30888a256136d9c550bf1ada77d6ea78a48af
  URL:=https://github.com/arut/nginx-dav-ext-module.git
  MIRROR_HASH:=70bb4c3907f4b783605500ba494e907aede11f8505702e370012abb3c177dc5b
  PROTO:=git
endef

define Download/nginx-mod-ubus
  VERSION:=b2d7260dcb428b2fb65540edb28d7538602b4a26
  URL:=https://github.com/Ansuel/nginx-ubus-module.git
  MIRROR_HASH:=472cef416d25effcac66c85417ab6596e634a7a64d45b709bb090892d567553c
  PROTO:=git
endef

define Module/Download
  define Download/nginx-mod-$(1) +=

    SUBDIR:=nginx-mod-$(1)
    FILE:=nginx-mod-$(1)-$$$$(VERSION).tar.xz
  endef
endef
$(foreach m,$(PKG_MOD_EXTRA),$(eval $(call Module/Download,$(m))))

define Module/nginx-mod-naxsi/install
	$(INSTALL_DIR) $(1)/etc/nginx
	$(INSTALL_CONF) $(PKG_BUILD_DIR)/nginx-mod-naxsi/naxsi_config/naxsi_core.rules $(1)/etc/nginx
endef

define Quilt/Refresh/Package
	$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/nginx,nginx/)
	$(foreach m,$(PKG_MOD_PATCHED),
		$(call Quilt/RefreshDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/nginx-mod-$(m),nginx-mod-$(m)/)
	)
endef

define Build/Patch
	$(if $(QUILT),rm -rf $(PKG_BUILD_DIR)/patches; mkdir -p $(PKG_BUILD_DIR)/patches)
	$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/nginx,nginx/)
	$(foreach m,$(PKG_MOD_PATCHED),$(if $(or $(CONFIG_PACKAGE_nginx-mod-$(m)),$(QUILT)),
		$(call PatchDir,$(PKG_BUILD_DIR),$(PATCH_DIR)/nginx-mod-$(m),nginx-mod-$(m)/)
	))
	$(if $(QUILT),touch $(PKG_BUILD_DIR)/.quilt_used)
endef

define Module/Build/Prepare
	$(eval $(call Download,nginx-mod-$(1)))
	$(eval $(Download/nginx-mod-$(1)))
	mkdir -p $(PKG_BUILD_DIR)/nginx-mod-$(1)
	xzcat $(DL_DIR)/$(FILE) | tar -C $(PKG_BUILD_DIR)/nginx-mod-$(1) $(TAR_OPTIONS) --strip-components 1
endef

define Build/Prepare
	rm -rf $(PKG_BUILD_DIR)
	mkdir -p $(PKG_BUILD_DIR)
	$(PKG_UNPACK)

	$(foreach m,$(filter-out $(PKG_MOD_PATCHED),$(PKG_MOD_EXTRA)),$(if $(CONFIG_PACKAGE_nginx-mod-$(m)),
		$(call Module/Build/Prepare,$(m))
	))
	$(foreach m,$(PKG_MOD_PATCHED),$(if $(or $(CONFIG_PACKAGE_nginx-mod-$(m)),$(QUILT)),
		$(call Module/Build/Prepare,$(m))
	))
	$(Build/Patch)
endef

# $(1) module name
# $(2) module additional dependency
# $(3) module so name (stripped of the finaly _module.so)
# $(4) module description
define BuildModule
  define Package/nginx-mod-$(1)
    $(call Package/nginx/default)
    DEPENDS:=+nginx-ssl $(2)
    TITLE:=Nginx $(1) module
  endef

  define Package/nginx-mod-$(1)/description
    $(strip $(4))
  endef

  define Package/nginx-mod-$(1)/install
	$(INSTALL_DIR) $$(1)/usr/lib/nginx/modules
	$(foreach m,$(3),
	  $(CP) $$(PKG_INSTALL_DIR)/usr/lib/nginx/modules/$(m)_module.so $$(1)/usr/lib/nginx/modules
	)
	$(call Module/nginx-mod-$(1)/install,$$(1))
  endef

  $$(eval $$(call BuildPackage,nginx-mod-$(1)))
endef

TARGET_CFLAGS += -DNGX_LUA_NO_BY_LUA_BLOCK

ifneq ($(CONFIG_PACKAGE_nginx-mod-lua),)
  CONFIGURE_VARS += LUAJIT_INC=$(STAGING_DIR)/usr/include/luajit-* \
					LUAJIT_LIB=$(STAGING_DIR)/usr/lib
endif

CONFIGURE_VARS += CONFIG_BIG_ENDIAN=$(CONFIG_BIG_ENDIAN)

CONFIGURE_ARGS += \
	--crossbuild=Linux::$(ARCH) \
	--prefix=/usr \
	--conf-path=/etc/nginx/nginx.conf \
	--modules-path=/usr/lib/nginx/modules \
	--error-log-path=stderr \
	--pid-path=/var/run/nginx.pid \
	--lock-path=/var/lock/nginx.lock \
	--http-log-path=/var/log/nginx/access.log \
	--http-client-body-temp-path=/var/lib/nginx/body \
	--http-proxy-temp-path=/var/lib/nginx/proxy \
	--http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
	--with-cc="$(TARGET_CC)" \
	--with-cc-opt="$(TARGET_CPPFLAGS) $(TARGET_CFLAGS)" \
	--with-ld-opt="$(TARGET_LDFLAGS)" \
	--without-http_upstream_zone_module \
	--without-pcre2 \
	--with-compat \
	--with-http_ssl_module \
	$(if $(call IsDisabled,NGINX_HTTP_CACHE),--without-http-cache) \
	$(if $(call IsDisabled,NGINX_PCRE),--without-pcre) \
	$(if $(call IsDisabled,NGINX_HTTP_CHARSET),--without-http_charset_module) \
	$(if $(call IsDisabled,NGINX_HTTP_GZIP),--without-http_gzip_module) \
	$(if $(call IsDisabled,NGINX_HTTP_SSI),--without-http_ssi_module) \
	$(if $(call IsDisabled,NGINX_HTTP_USERID),--without-http_userid_module) \
	$(if $(call IsDisabled,NGINX_HTTP_ACCESS),--without-http_access_module) \
	$(if $(call IsDisabled,NGINX_HTTP_AUTH_BASIC),--without-http_auth_basic_module) \
	$(if $(call IsDisabled,NGINX_HTTP_AUTOINDEX),--without-http_autoindex_module) \
	$(if $(call IsDisabled,NGINX_HTTP_GEO),--without-http_geo_module) \
	$(if $(call IsDisabled,NGINX_HTTP_MAP),--without-http_map_module) \
	$(if $(call IsDisabled,NGINX_HTTP_SPLIT_CLIENTS),--without-http_split_clients_module) \
	$(if $(call IsDisabled,NGINX_HTTP_REFERER),--without-http_referer_module) \
	$(if $(call IsDisabled,NGINX_HTTP_REWRITE),--without-http_rewrite_module) \
	$(if $(call IsDisabled,NGINX_HTTP_PROXY),--without-http_proxy_module) \
	$(if $(call IsDisabled,NGINX_HTTP_FASTCGI),--without-http_fastcgi_module) \
	$(if $(call IsDisabled,NGINX_HTTP_UWSGI),--without-http_uwsgi_module) \
	$(if $(call IsDisabled,NGINX_HTTP_SCGI),--without-http_scgi_module) \
	$(if $(call IsDisabled,NGINX_HTTP_MEMCACHED),--without-http_memcached_module) \
	$(if $(call IsDisabled,NGINX_HTTP_LIMIT_CONN),--without-http_limit_conn_module) \
	$(if $(call IsDisabled,NGINX_HTTP_LIMIT_REQ),--without-http_limit_req_module) \
	$(if $(call IsDisabled,NGINX_HTTP_EMPTY_GIF),--without-http_empty_gif_module) \
	$(if $(call IsDisabled,NGINX_HTTP_BROWSER),--without-http_browser_module) \
	$(if $(call IsDisabled,NGINX_HTTP_UPSTREAM_HASH),--without-http_upstream_hash_module) \
	$(if $(call IsDisabled,NGINX_HTTP_UPSTREAM_IP_HASH),--without-http_upstream_ip_hash_module) \
	$(if $(call IsDisabled,NGINX_HTTP_UPSTREAM_LEAST_CONN),--without-http_upstream_least_conn_module) \
	$(if $(call IsDisabled,NGINX_HTTP_UPSTREAM_KEEPALIVE),--without-http_upstream_keepalive_module) \
	$(if $(call IsEnabled,IPV6),--with-ipv6) \
	$(if $(call IsEnabled,NGINX_HTTP_GZIP_STATIC),--with-http_gzip_static_module) \
	$(if $(call IsEnabled,NGINX_STUB_STATUS),--with-http_stub_status_module) \
	$(if $(call IsEnabled,NGINX_FLV),--with-http_flv_module) \
	$(if $(call IsEnabled,NGINX_DAV),--with-http_dav_module) \
	$(if $(call IsEnabled,NGINX_HTTP_AUTH_REQUEST),--with-http_auth_request_module) \
	$(if $(call IsEnabled,NGINX_HTTP_V2),--with-http_v2_module) \
	$(if $(call IsEnabled,NGINX_HTTP_REAL_IP),--with-http_realip_module) \
	$(if $(call IsEnabled,NGINX_HTTP_SECURE_LINK),--with-http_secure_link_module) \
	$(if $(call IsEnabled,NGINX_HTTP_SUB),--with-http_sub_module) \
	$(if $(CONFIG_PACKAGE_nginx-mod-stream),--with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module) \
	$(if $(CONFIG_PACKAGE_nginx-mod-naxsi),--add-dynamic-module=$(PKG_BUILD_DIR)/nginx-mod-naxsi/naxsi_src) \
	$(foreach m,$(filter-out naxsi,$(PKG_MOD_EXTRA)), \
		$(if $(CONFIG_PACKAGE_nginx-mod-$(m)),--add-dynamic-module=$(PKG_BUILD_DIR)/nginx-mod-$(m)))

$(eval $(call BuildPackage,nginx-ssl))
$(eval $(call BuildPackage,nginx-full))
$(eval $(call BuildPackage,nginx-mod-luci))
$(eval $(call BuildModule,stream,+@NGINX_STREAM_CORE_MODULE, \
	ngx_stream, Add support for NGINX request streaming.))
$(eval $(call BuildModule,lua,+luajit,ngx_http_lua, \
	Enable Lua module))
$(eval $(call BuildModule,ubus,+libubus +libjson-c +libblobmsg-json +@NGINX_UBUS, \
	ngx_http_ubus,Enable UBUS api support directly from the server.))
$(eval $(call BuildModule,dav-ext,+@NGINX_DAV +libxml2,ngx_http_dav_ext, \
	Enable the WebDAV methods PROPFIND OPTIONS LOCK UNLOCK.))
$(eval $(call BuildModule,headers-more,,ngx_http_headers_more_filter, \
	Set and clear input and output headers...more than "add"!))
$(eval $(call BuildModule,rtmp,,ngx_rtmp, \
	Add support for NGINX-based Media Streaming Server module. DASH enhanced))
$(eval $(call BuildModule,ts,,ngx_http_ts, \
	Add support for MPEG-TS Live Module module.))
$(eval $(call BuildModule,brotli,,ngx_http_brotli_filter ngx_http_brotli_static, \
	Add support for brotli compression module.))
$(eval $(call BuildModule,naxsi,,ngx_http_naxsi, \
	Enable NAXSI module.))

# TODO: remove after a transition period (together with pkg nginx-util):
# It is for smoothly substituting nginx and nginx-mod-luci-ssl (by nginx-ssl
# respectively nginx-mod-luci). Add above commented PROVIDES when removing.

define Package/nginx
  TITLE:=Dummy package for transition when upgrading.
  DEPENDS:=+nginx-ssl
  PKGARCH:=all
endef

define Package/nginx/install
	$(INSTALL_DIR) $(1)/usr/bin
endef

$(eval $(call BuildPackage,nginx))

define Package/nginx-mod-luci-ssl
  TITLE:=Dummy package for transition when upgrading.
  DEPENDS:=+nginx-mod-luci
  PKGARCH:=all
endef

define Package/nginx-mod-luci-ssl/install
	$(INSTALL_DIR) $(1)/usr/bin
endef

$(eval $(call BuildPackage,nginx-mod-luci-ssl))
