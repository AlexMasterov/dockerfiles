FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-nginx-tarantool" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.13.6
ARG NGINX_CONFIG=/etc/nginx

ARG LUAJIT_BRANCH=tarantool-1.7
ARG LUAROCKS_TAG=2.4.3

ARG DEVELKIT_MODULE_TAG=v0.3.0
ARG LUA_MODULE_TAG=v0.10.11
ARG TARANTOOL_UPSTREAM_TAG=v2.5-stable

ENV LUAJIT_LIB=/usr/local/lib \
    LUAJIT_INC=/usr/local/include/luajit-2.1

RUN set -x \
  && apk add --update \
    tini \
  && addgroup -g 82 -S nginx \
  && adduser -u 82 -S -D -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN set -x \
  && apk add --virtual .nginx-build-dependencies \
    cmake \
    curl \
    g++ \
    gcc \
    git \
    libressl-dev \
    linux-headers \
    make \
    pcre-dev \
    zlib-dev \
  && : "---------- Build flags ----------" \
  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  && export CFLAGS="-O2 -march=native -fstack-protector-strong -fpic -fpie" \
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \
  && : "---------- LuaJIT ----------" \
  && apk add --virtual .luajit-runtime-dependencies \
    libgcc \
  && git clone -b ${LUAJIT_BRANCH} --depth 1 https://github.com/tarantool/luajit.git /tmp/luajit \
  && cd /tmp/luajit \
  && make \
    CFLAGS+="-fPIC" \
    XCFLAGS+="-DLUAJIT_ENABLE_GC64" \
  && make install \
  && ln -sf /usr/local/bin/luajit-2.1.0-beta3 /usr/local/bin/lua \
  && : "---------- LuaRocks ----------" \
  && git clone -b ${LUAROCKS_TAG} --depth 1 https://github.com/luarocks/luarocks.git /tmp/luarocks \
  && cd /tmp/luarocks \
  && ./configure \
    --with-lua-bin=/usr/local/bin \
    --with-lua-include=/usr/local/include/luajit-2.1 \
  && make build \
  && make install \
  && : "---------- Nginx Development Kit ----------" \
  && DEVELKIT_MODULE_DIR="/tmp/devel-kit" \
  && DEVELKIT_MODULE_GIT="https://github.com/simpl/ngx_devel_kit.git" \
  && git clone -o ${DEVELKIT_MODULE_TAG} --depth 1 ${DEVELKIT_MODULE_GIT} ${DEVELKIT_MODULE_DIR} \
  && : "---------- Nginx Lua Module  ----------" \
  && LUA_MODULE_DIR="/tmp/lua-module" \
  && LUA_MODULE_GIT="https://github.com/openresty/lua-nginx-module.git" \
  && git clone -o ${LUA_MODULE_TAG} --depth 1 ${LUA_MODULE_GIT} ${LUA_MODULE_DIR} \
  && : "---------- Nginx Tarantool Module  ----------" \
  && TARANTOOL_UPSTREAM_GIT="https://github.com/tarantool/nginx_upstream_module.git" \
  && TARANTOOL_UPSTREAM_DIR="/tmp/nginx_upstream_module" \
  && git clone --depth 1 ${TARANTOOL_UPSTREAM_GIT} ${TARANTOOL_UPSTREAM_DIR} \
  && git clone --depth 1 https://github.com/lloyd/yajl.git /tmp/yajl \
  && cd /tmp/yajl \
  && cmake \
    -DCMAKE_BUILD_TYPE=Release \
    . \
  && make \
  && make install \
  && : "---------- Msgpuck  ----------" \
  && git clone --depth 1 https://github.com/rtsisyk/msgpuck.git /tmp/msgpuck \
  && cd /tmp/msgpuck \
  && cmake \
    -DCMAKE_BUILD_TYPE=Release \
    . \
  && make \
  && make install \
  && : "---------- Nginx  ----------" \
  && NGINX_SOURCE="https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${NGINX_SOURCE} | tar xz -C /tmp \
  && cd /tmp/nginx-${NGINX_VERSION} \
  && ./configure \
    --prefix=${NGINX_CONFIG} \
    --conf-path=${NGINX_CONFIG}/nginx.conf \
    --modules-path=/usr/lib/nginx/modules \
    --sbin-path=/usr/sbin/nginx \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --without-http_autoindex_module \
    --without-http_ssi_module \
    --without-mail_imap_module \
    --without-mail_pop3_module \
    --without-mail_smtp_module \
    --with-file-aio \
    --with-http_auth_request_module \
    --with-http_dav_module \
    --with-http_degradation_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_realip_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_v2_module \
    --with-pcre \
    --with-pcre-jit \
    --with-stream \
    --with-stream_ssl_module  \
    --with-threads \
    --with-cc-opt="-flto" \
    --add-module=${DEVELKIT_MODULE_DIR} \
    --add-module=${LUA_MODULE_DIR} \
  && make \
  && make install \
  && runtimeDeps="$( \
		scanelf --needed --nobanner --recursive /usr/sbin/nginx /usr/lib/nginx/modules/*.so \
			| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
	)" \
  && apk add --virtual .nginx-runtime-dependencies \
    ${runtimeDeps} \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .nginx-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["nginx"]
