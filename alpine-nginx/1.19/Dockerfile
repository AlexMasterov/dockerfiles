FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-nginx:1.19" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.19.8
ARG NGINX_CONFIG=/etc/nginx

ARG NGINX_EXTRA_CCFLAGS="-march=native -O2 -fstack-protector-strong -flto -fpic -fpie"
ARG NGINX_EXTRA_LDFLAGS="-O2 -pie"

ARG SSL_LIB=OPENSSL

ARG HTTP_DEGRADATION=NO
ARG HTTP_GUNZIP=NO
ARG HTTP_GZIP=YES
ARG HTTP_GZIP_STATIC=NO
ARG HTTP_REFERER=NO
ARG HTTP_SECURE_LINK=NO
ARG HTTP_SLICE=NO
ARG HTTP_SPLIT_CLIENTS=NO
ARG HTTP_SSL=YES
ARG HTTP_STREAM=YES
ARG HTTP_STREAM_SSL=NO
ARG HTTP_V2=YES

ARG BROTLI=NO
ARG BROTLI_VERSION=1.0.0rc

ARG HEADERS_MORE=NO
ARG HEADERS_MORE_VERSION=0.33

ARG NJS=NO
ARG NJS_VERSION=0.5.2

ARG LUA=NO
ARG LUA_VERSION=0.10.19
ARG LUA_JIT_VERSION=2.1-20201229
ARG LUA_ROCKS=NO
ARG LUA_ROCKS_VERSION=3.5.0
ARG DEVEL_KIT_VERSION=0.3.1

ARG TARANTOOL_UPSTREAM=NO
ARG TARANTOOL_UPSTREAM_VERSION=2.7.1
ARG YAJL_VERSION=2.1.0
ARG MSGPUCK_VERSION=2.0

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S nginx; \
  adduser -u 82 -S -D -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN set -eux; \
  if_eq() { [ ${1} = ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xz} -C /tmp; }; \
  apk add --no-cache --virtual .nginx-build \
    g++ \
    gcc \
    linux-headers \
    make \
    pcre-dev \
    $(if_eq ${SSL_LIB}            'OPENSSL'  openssl-dev) \
    $(if_eq ${SSL_LIB}            'LIBRESSL' libressl-dev) \
    $(if_eq ${HTTP_GZIP}          'YES'      zlib-dev) \
    $(if_eq ${BROTLI}             'YES'      brotli-dev) \
    $(if_eq ${TARANTOOL_UPSTREAM} 'YES'      cmake); \
  : "-- Setup build flags"; \
  export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)"; \
  : "-- Setup modules"; \
  if [ ${BROTLI} = 'YES' ]; then \
    load_archive "https://github.com/google/ngx_brotli/archive/v${BROTLI_VERSION}.tar.gz"; \
  fi; \
  if [ ${HEADERS_MORE} = 'YES' ]; then \
    load_archive "https://github.com/openresty/headers-more-nginx-module/archive/v${HEADERS_MORE_VERSION}.tar.gz"; \
  fi; \
  if [ ${NJS} = 'YES' ]; then \
    load_archive "https://github.com/nginx/njs/archive/${NJS_VERSION}.tar.gz"; \
  fi; \
  if [ ${LUA} = 'YES' ]; then \
    load_archive "https://github.com/openresty/lua-nginx-module/archive/v${LUA_VERSION}.tar.gz"; \
    load_archive "https://github.com/vision5/ngx_devel_kit/archive/v${DEVEL_KIT_VERSION}.tar.gz"; \
    load_archive "https://github.com/openresty/luajit2/archive/v${LUA_JIT_VERSION}.tar.gz"; \
    export \
      LUAJIT_BIN="/usr/local/bin" \
      LUAJIT_LIB="/usr/local/lib" \
      LUAJIT_INC="/usr/local/include"; \
    cd /tmp/luajit2-${LUA_JIT_VERSION}; \
    make; \
    make install \
      INSTALL_BIN="${LUAJIT_BIN}" \
      INSTALL_LIB="${LUAJIT_LIB}" \
      INSTALL_INC="${LUAJIT_INC}" \
      INSTALL_MAN="/luajit2"; \
    if [ ${LUA_ROCKS} = 'YES' ]; then \
      load_archive "https://github.com/luarocks/luarocks/archive/v${LUA_ROCKS_VERSION}.tar.gz"; \
      cd /tmp/luarocks-${LUA_ROCKS_VERSION}; \
      ./configure \
        --with-lua-bin="${LUAJIT_BIN}" \
        --with-lua-lib="${LUAJIT_LIB}" \
        --with-lua-include="${LUAJIT_INC}" \
        --force-config \
        --disable-incdir-check; \
      make build; \
      make install; \
    fi; \
  fi; \
  if [ ${TARANTOOL_UPSTREAM} = 'YES' ]; then \
    load_archive "https://github.com/tarantool/nginx_upstream_module/archive/v${TARANTOOL_UPSTREAM_VERSION}.tar.gz"; \
    : "-- Build msgpuck"; \
    load_archive "https://github.com/rtsisyk/msgpuck/archive/${MSGPUCK_VERSION}.tar.gz"; \
    cd /tmp/msgpuck-${MSGPUCK_VERSION}; \
    cmake -DCMAKE_BUILD_TYPE=Release .; \
    make; \
    : "-- Build yajl"; \
    load_archive "https://github.com/lloyd/yajl/archive/${YAJL_VERSION}.tar.gz"; \
    cd /tmp/yajl-${YAJL_VERSION}; \
    cmake -DCMAKE_BUILD_TYPE=Release .; \
    make yajl yajl_s; \
    : "-- Setup Nginx compiler flags for Tarantool upstream"; \
    MSGPUCK_DIR="/tmp/msgpuck-${MSGPUCK_VERSION}"; \
    YAJL_DIR="/tmp/yajl-${YAJL_VERSION}/yajl-${YAJL_VERSION}"; \
    NGINX_EXTRA_CCFLAGS="${NGINX_EXTRA_CCFLAGS} -I${MSGPUCK_DIR} -I${YAJL_DIR}/include"; \
    NGINX_EXTRA_LDFLAGS="${NGINX_EXTRA_LDFLAGS} -L${MSGPUCK_DIR} -L${YAJL_DIR}/lib";\
  fi; \
  : "-- Download Nginx"; \
  load_archive "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz"; \
  : "-- Build Nginx"; \
  cd /tmp/nginx-${NGINX_VERSION}; \
  ./configure \
    --prefix="${NGINX_CONFIG}" \
    --conf-path="${NGINX_CONFIG}/nginx.conf" \
    --modules-path="/usr/lib/nginx/modules" \
    --sbin-path="/usr/sbin/nginx" \
    --pid-path="/var/run/nginx.pid" \
    --lock-path="/var/run/nginx.lock" \
    --error-log-path="/var/log/nginx/error.log" \
    --http-log-path="/var/log/nginx/access.log" \
    --http-client-body-temp-path="/var/cache/nginx/client_temp" \
    --http-proxy-temp-path="/var/cache/nginx/proxy_temp" \
    --http-fastcgi-temp-path="/var/cache/nginx/fastcgi_temp" \
    --http-uwsgi-temp-path="/var/cache/nginx/uwsgi_temp" \
    --http-scgi-temp-path="/var/cache/nginx/scgi_temp" \
    --user="nginx" \
    --group="nginx" \
    --without-http_auth_basic_module \
    --without-http_autoindex_module \
    --without-http_empty_gif_module \
    --without-http_geo_module \
    --without-http_grpc_module \
    --without-http_memcached_module \
    --without-http_scgi_module \
    --without-http_ssi_module \
    --without-http_uwsgi_module \
    --without-mail_imap_module \
    --without-mail_pop3_module \
    --without-mail_smtp_module \
    --with-http_realip_module \
    --with-pcre \
      --with-pcre-jit \
    --with-threads \
    --with-file-aio \
    --with-cc-opt="-D FD_SETSIZE=$(ulimit -n) ${NGINX_EXTRA_CCFLAGS}" \
    --with-ld-opt="${NGINX_EXTRA_LDFLAGS}" \
    $([ ${HTTP_STREAM} = 'YES' ] && echo \
      --with-stream \
      $(if_eq ${HTTP_STREAM_SSL}  'YES' --with-stream_ssl_module) \
    \ ) \
    $(if_eq ${HTTP_GZIP}          'NO'    --without-http_gzip_module) \
    $(if_eq ${HTTP_REFERER}       'NO'    --without-http_referer_module) \
    $(if_eq ${HTTP_SPLIT_CLIENTS} 'NO'    --without-http_split_clients_module) \
    $(if_eq ${HTTP_V2}            'YES'   --with-http_v2_module) \
    $(if_eq ${HTTP_SSL}           'YES'   --with-http_ssl_module) \
    $(if_eq ${HTTP_GUNZIP}        'YES'   --with-http_gunzip_module) \
    $(if_eq ${HTTP_GZIP_STATIC}   'YES'   --with-http_gzip_static_module) \
    $(if_eq ${HTTP_DEGRADATION}   'YES'   --with-http_degradation_module) \
    $(if_eq ${HTTP_SECURE_LINK}   'YES'   --with-http_secure_link_module) \
    $(if_eq ${HTTP_SLICE}         'YES'   --with-http_slice_module) \
    $(if_eq ${BROTLI}             'YES'   --add-module="/tmp/ngx_brotli-${BROTLI_VERSION}") \
    $(if_eq ${HEADERS_MORE}       'YES'   --add-module="/tmp/headers-more-nginx-module-${HEADERS_MORE_VERSION}") \
    $(if_eq ${NJS}                'YES'   --add-module="/tmp/njs-${NJS_VERSION}/nginx") \
    $(if_eq ${TARANTOOL_UPSTREAM} 'YES'   --add-module="/tmp/nginx_upstream_module-${TARANTOOL_UPSTREAM_VERSION}") \
    $(if_eq ${LUA}                'YES'  " \
      --add-module=/tmp/ngx_devel_kit-${DEVEL_KIT_VERSION} \
      --add-module=/tmp/lua-nginx-module-${LUA_VERSION}" \
    ); \
  make; \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/sbin/nginx \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .nginx-runtime ${runtime_deps}; \
  : "-- Strip files"; \
  find \
    /usr/bin \
    /usr/sbin \
    /usr/local/bin \
    -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .nginx-build; \
  rm -rf /tmp/* /var/tmp/*

STOPSIGNAL SIGQUIT

CMD ["nginx"]
