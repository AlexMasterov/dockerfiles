FROM alpine:3.14

LABEL repository.hub="alexmasterov/alpine-nginx" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.21.2

ARG NGINX_CONFIG=/etc/nginx
ARG NGINX_EXTRA_CCFLAGS="-O2 -fstack-protector-strong -flto -fPIC -fpie"
ARG NGINX_EXTRA_LDFLAGS="-O2 -pie"

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
ARG HTTP_STREAM_SSL=YES
ARG HTTP_V2=YES

ARG BROTLI=NO
ARG BROTLI_VERSION=1.0.0rc

ARG HEADERS_MORE=NO
ARG HEADERS_MORE_VERSION=0.33

ARG NJS=NO
ARG NJS_VERSION=0.6.2

ARG LUA=NO
ARG LUA_VERSION=0.10.20rc1
ARG LUA_JIT_VERSION=2.1-20210510
ARG LUA_ROCKS=NO
ARG LUA_ROCKS_VERSION=3.7.0
ARG DEVEL_KIT_VERSION=0.3.1

ARG TARANTOOL_UPSTREAM=NO
ARG TARANTOOL_UPSTREAM_VERSION=2.7.1
ARG MSGPUCK_VERSION=2.0
ARG YAJL_VERSION=2.1.0

ARG USE_TZ=YES
ARG LIB_SSL=OPENSSL

ARG OUT_STRIP=YES

RUN set -eux; \
  contains() { if [ -z ${1##*${2}*} ]; then shift 2; echo ${*:-}; fi; }; \
  get_deps() { find ${*:-} -type f -and \( -executable -or -name '*.so*' \) -exec ldd '{}' ';' | sed -rn "s/\s*(.* => )?(.*) \(.*/\2/p" | sort -u; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo ${*:-}; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo ${*:-}; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  to_lowercase() { echo ${1:-} | tr '[:upper:]' '[:lower:]'; }; \
  \
  apk add --no-cache --virtual .nginx-build \
    g++ \
    gcc \
    linux-headers \
    make \
    pcre-dev \
    $(contains "${HTTP_SSL}${HTTP_STREAM_SSL}" 'YES' $(to_lowercase ${LIB_SSL})-dev) \
    \
    $(if_yes ${HTTP_GZIP} zlib-dev) \
    $(if_yes ${BROTLI}    brotli-dev) \
    $(if_yes ${TARANTOOL_UPSTREAM} \
      cmake \
      yajl-dev \
    ) \
    ; \
  if [ ${USE_TZ} = 'YES' ]; then \
    apk add --no-cache --virtual .nginx-tz tzdata; \
  fi; \
  : "-- make: setup flags"; \
  export MAKEFLAGS="-j $(( $(getconf _NPROCESSORS_ONLN) - 1 ))"; \
  \
  : "-- Nginx: setup modules"; \
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
      LUAJIT_BIN=/usr/local/bin \
      LUAJIT_LIB=/usr/local/lib \
      LUAJIT_INC=/usr/local/include; \
    cd /tmp/luajit2-${LUA_JIT_VERSION}; \
    make; \
    make install \
      INSTALL_BIN=${LUAJIT_BIN} \
      INSTALL_LIB=${LUAJIT_LIB} \
      INSTALL_INC=${LUAJIT_INC} \
      INSTALL_MAN=/luajit2; \
    \
    if [ ${LUA_ROCKS} = 'YES' ]; then \
      load_archive "https://github.com/luarocks/luarocks/archive/v${LUA_ROCKS_VERSION}.tar.gz"; \
      cd /tmp/luarocks-${LUA_ROCKS_VERSION}; \
      ./configure \
        --with-lua-bin=${LUAJIT_BIN} \
        --with-lua-lib=${LUAJIT_LIB} \
        --with-lua-include=${LUAJIT_INC} \
        --force-config \
        --disable-incdir-check; \
      make luarocks; \
      make install; \
    fi; \
  fi; \
  if [ ${TARANTOOL_UPSTREAM} = 'YES' ]; then \
    load_archive "https://github.com/tarantool/nginx_upstream_module/archive/v${TARANTOOL_UPSTREAM_VERSION}.tar.gz"; \
    : "-- msgpuck: build"; \
    load_archive "https://github.com/rtsisyk/msgpuck/archive/${MSGPUCK_VERSION}.tar.gz"; \
    cd /tmp/msgpuck-${MSGPUCK_VERSION}; \
    sed -i "s/if (NOT \${PROJECT_SOURCE_DIR}.*)/if (TRUE)/" CMakeLists.txt; \
    cmake -DCMAKE_BUILD_TYPE=Release .; \
    make msgpuck; \
    : "-- Nginx: setup compiler flags for Tarantool upstream module"; \
    MSGPUCK_DIR=/tmp/msgpuck-${MSGPUCK_VERSION}; \
    NGINX_EXTRA_CCFLAGS="-I${MSGPUCK_DIR} ${NGINX_EXTRA_CCFLAGS:-}"; \
    NGINX_EXTRA_LDFLAGS="-L${MSGPUCK_DIR} ${NGINX_EXTRA_LDFLAGS:-}"; \
  fi; \
  : "-- Nginx: download source"; \
  load_archive "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz"; \
  : "-- Nginx: build"; \
  cd /tmp/nginx-${NGINX_VERSION}; \
  ./configure \
    --prefix=${NGINX_CONFIG} \
    --conf-path="${NGINX_CONFIG}/nginx.conf" \
    --sbin-path="/usr/local/bin/nginx" \
    --modules-path="/usr/local/lib/nginx/modules" \
    --pid-path="/var/run/nginx/nginx.pid" \
    --lock-path="/var/run/nginx/nginx.lock" \
    --error-log-path="/var/log/nginx/error.log" \
    --http-log-path="/var/log/nginx/access.log" \
    --http-proxy-temp-path="/var/cache/nginx/proxy_temp" \
    --http-client-body-temp-path="/var/cache/nginx/client_temp" \
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
    --with-cc-opt="-D FD_SETSIZE=$(ulimit -n) ${NGINX_EXTRA_CCFLAGS:-}" \
    --with-ld-opt="${NGINX_EXTRA_LDFLAGS:-}" \
    $(if_yes ${HTTP_STREAM} \
      --with-stream \
      $(if_yes ${HTTP_STREAM_SSL} --with-stream_ssl_module) \
    ) \
    $(if_no ${HTTP_GZIP}           --without-http_gzip_module) \
    $(if_no ${HTTP_REFERER}        --without-http_referer_module) \
    $(if_no ${HTTP_SPLIT_CLIENTS}  --without-http_split_clients_module) \
    $(if_yes ${HTTP_V2}            --with-http_v2_module) \
    $(if_yes ${HTTP_SSL}           --with-http_ssl_module) \
    $(if_yes ${HTTP_GUNZIP}        --with-http_gunzip_module) \
    $(if_yes ${HTTP_GZIP_STATIC}   --with-http_gzip_static_module) \
    $(if_yes ${HTTP_DEGRADATION}   --with-http_degradation_module) \
    $(if_yes ${HTTP_SECURE_LINK}   --with-http_secure_link_module) \
    $(if_yes ${HTTP_SLICE}         --with-http_slice_module) \
    $(if_yes ${BROTLI}             --add-module="/tmp/ngx_brotli-${BROTLI_VERSION}") \
    $(if_yes ${HEADERS_MORE}       --add-module="/tmp/headers-more-nginx-module-${HEADERS_MORE_VERSION}") \
    $(if_yes ${NJS}                --add-module="/tmp/njs-${NJS_VERSION}/nginx") \
    $(if_yes ${TARANTOOL_UPSTREAM} --add-module="/tmp/nginx_upstream_module-${TARANTOOL_UPSTREAM_VERSION}") \
    $(if_yes ${LUA} \
      --add-module="/tmp/ngx_devel_kit-${DEVEL_KIT_VERSION}" \
      --add-module="/tmp/lua-nginx-module-${LUA_VERSION}" \
    ); \
  make build; \
  make install; \
  \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    : "-- Out: strip executables"; \
    find /usr/local -type f -executable \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  runtime_deps=$(get_deps /usr/local/bin /usr/local/lib | sed -rn "s/\/usr\/lib\/(.*)/so:\1/p"); \
  apk add --no-cache --virtual .nginx-runtime ${runtime_deps}; \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .nginx-build; \
  rm -rf /tmp/* /var/tmp/*

STOPSIGNAL SIGQUIT

CMD ["nginx"]
