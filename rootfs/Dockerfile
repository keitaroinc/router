FROM quay.io/deis/base:0.2.0

RUN adduser --system \
	--shell /bin/bash \
	--disabled-password \
	--home /opt/router \
	--group \
	router

COPY . /

RUN apt-get update \
	&& apt-get install -y \
		gcc \
		libgeoip1 \
		libgeoip-dev \
		libssl-dev \
		libpcre3-dev \
		make \
		unzip \
	&& export BUILD_PATH=/tmp/build \
	&& mkdir "$BUILD_PATH" \
	&& cd "$BUILD_PATH" \
	&& export LUAJIT_VERSION=2.1.0-beta2 LUAJIT_PREFIX=/opt/luajit LUAJIT_LIB=/opt/luajit/lib LUAJIT_INC=/opt/luajit/include/luajit-2.1 LUAROCKS_VERSION=2.3.0 \
	&& get_src 713924ca034b9d99c84a0b7b701794c359ffb54f8e3aa2b84fad52d98384cf47 \
		"http://luajit.org/download/LuaJIT-$LUAJIT_VERSION.tar.gz" \
	&& get_src 68e38feeb66052e29ad1935a71b875194ed8b9c67c2223af5f4d4e3e2464ed97 \
		"http://keplerproject.github.io/luarocks/releases/luarocks-$LUAROCKS_VERSION.tar.gz" \
	&& cd "$BUILD_PATH/LuaJIT-$LUAJIT_VERSION" \
	&& make PREFIX=$LUAJIT_PREFIX \
	&& make install PREFIX=$LUAJIT_PREFIX \
	&& cd "$BUILD_PATH/luarocks-$LUAROCKS_VERSION" \
	&& ./configure \
	  --prefix="$LUAJIT_PREFIX" \
	  --with-lua="$LUAJIT_PREFIX" \
	  --lua-suffix="jit-$LUAJIT_VERSION" \
	  --with-lua-include="$LUAJIT_INC" \
	&& make build \
	&& make install \
	&& $LUAJIT_PREFIX/bin/luarocks install lua-resty-auto-ssl \
	&& mkdir /etc/resty-auto-ssl \
	&& chown router:router /etc/resty-auto-ssl \
	&& export NGINX_VERSION=1.10.1 SIGNING_KEY=A1C052F8 VTS_VERSION=0.1.8 NDK_VERSION=0.3.0 NGXLUA_VERSION=0.10.6 PREFIX=/opt/router \
	&& cd "$BUILD_PATH" \
	&& rm -rf "$PREFIX" \
	&& mkdir "$PREFIX" \
	&& get_src_gpg $SIGNING_KEY \
		"http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz" \
	&& get_src 6bb9a36d8d70302d691c49557313fb7262cafd942a961d11a2730d9a5d9f70e0 \
		"https://github.com/vozlt/nginx-module-vts/archive/v$VTS_VERSION.tar.gz" \
	&& get_src 88e05a99a8a7419066f5ae75966fb1efc409bad4522d14986da074554ae61619 \
		"https://github.com/simpl/ngx_devel_kit/archive/v$NDK_VERSION.tar.gz" \
	&& get_src b98c4f648589bbf3e2c3d5fd18664e7a7ef89ac083e96c8e984fa919e7a7c073 \
		"https://github.com/openresty/lua-nginx-module/archive/v$NGXLUA_VERSION.tar.gz" \
	&& cd "$BUILD_PATH/nginx-$NGINX_VERSION" \
	&& ./configure \
	  --prefix="$PREFIX" \
	  --pid-path=/tmp/nginx.pid \
	  --with-debug \
	  --with-pcre-jit \
	  --with-ipv6 \
	  --with-http_ssl_module \
	  --with-http_stub_status_module \
	  --with-http_realip_module \
	  --with-http_auth_request_module \
	  --with-http_addition_module \
	  --with-http_dav_module \
	  --with-http_geoip_module \
	  --with-http_gzip_static_module \
	  --with-http_sub_module \
	  --with-http_v2_module \
	  --with-mail \
	  --with-mail_ssl_module \
	  --with-stream \
	  --with-ld-opt="-Wl,-rpath,$LUAJIT_LIB" \
	  --add-module="$BUILD_PATH/nginx-module-vts-$VTS_VERSION" \
	  --add-module="$BUILD_PATH/ngx_devel_kit-$NDK_VERSION" \
	  --add-module="$BUILD_PATH/lua-nginx-module-$NGXLUA_VERSION" \
	&& make \
	&& make install \
		&& apt-get remove -y --auto-remove --purge \
		gcc \
		libgeoip-dev \
		libssl-dev \
		libpcre3-dev \
		make \
		unzip \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/man /usr/share/doc \
	&& rm -rf "$BUILD_PATH"

# Re-copy these files because the previous step will have overwritten them
COPY opt/router /opt/router

# Fix some permissions since we'll be running as a non-root user
RUN chown -R router:router /opt/router

USER router

CMD ["/opt/router/sbin/boot"]
EXPOSE 2222 8080 6443 9090
