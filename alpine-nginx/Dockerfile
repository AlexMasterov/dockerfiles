FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-nginx" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.19.8
ARG NGINX_CONFIG=/etc/nginx

ARG HTTP_DEGRADATION=NO
ARG HTTP_REFERER=NO
ARG HTTP_SECURE_LINK=NO
ARG HTTP_SLICE=NO
ARG HTTP_SPLIT_CLIENTS=NO

ARG NJS=NO
ARG NJS_VERSION=0.5.2

ARG BROTLI=NO
ARG BROTLI_VERSION=1.0.0rc

ARG HEADERS_MORE=NO
ARG HEADERS_MORE_VERSION=0.33

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S nginx; \
  adduser -u 82 -S -D -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN set -eux; \
  if_option() { [ ${1} = ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar xz -C /tmp; }; \
  apk add --no-cache --virtual .nginx-build \
    g++ \
    gcc \
    linux-headers \
    make \
    openssl-dev \
    pcre-dev \
    zlib-dev \
    $(if_option ${BROTLI} 'YES' brotli-dev) \
    ; \
  if [ ${NJS} = 'YES' ]; then \
    load_archive "https://github.com/nginx/njs/archive/${NJS_VERSION}.tar.gz"; \
  fi; \
  if [ ${BROTLI} = 'YES' ]; then \
    load_archive "https://github.com/google/ngx_brotli/archive/v${BROTLI_VERSION}.tar.gz"; \
  fi; \
  if [ ${HEADERS_MORE} = 'YES' ]; then \
    load_archive "https://github.com/openresty/headers-more-nginx-module/archive/v${HEADERS_MORE_VERSION}.tar.gz"; \
  fi; \
  : "-- Download Nginx"; \
  load_archive "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz"; \
  : "-- Setup build flags"; \
  export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)"; \
  : "-- Build Nginx"; \
  cd /tmp/nginx-${NGINX_VERSION}; \
  ./configure \
    --prefix=${NGINX_CONFIG} \
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
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_realip_module \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-pcre \
      --with-pcre-jit \
    --with-stream \
      --with-stream_ssl_module  \
    --with-threads \
    --with-file-aio \
    --with-ld-opt="-O2 -pie" \
    --with-cc-opt="-O2 -march=native -fstack-protector-strong -flto -fpic -fpie -D FD_SETSIZE=$(ulimit -n)" \
    $(if_option ${HTTP_REFERER}       'NO'  --without-http_referer_module) \
    $(if_option ${HTTP_SPLIT_CLIENTS} 'NO'  --without-http_split_clients_module) \
    $(if_option ${HTTP_DEGRADATION}   'YES' --with-http_degradation_module) \
    $(if_option ${HTTP_SECURE_LINK}   'YES' --with-http_secure_link_module) \
    $(if_option ${HTTP_SLICE}         'YES' --with-http_slice_module) \
    $(if_option ${NJS}                'YES' --add-module="/tmp/njs-${NJS_VERSION}/nginx") \
    $(if_option ${BROTLI}             'YES' --add-module="/tmp/ngx_brotli-${BROTLI_VERSION}") \
    $(if_option ${HEADERS_MORE}       'YES' --add-module="/tmp/headers-more-nginx-module-${HEADERS_MORE_VERSION}") \
    ; \
  make; \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/sbin/nginx \
      | tr ',' '\n' \
      | sort -u \
      | sed -e 's/^/so:\1/' \
    )"; \
  apk add --no-cache --virtual .nginx-runtime ${runtime_deps}; \
  : "-- Strip files"; \
  find /usr/sbin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .nginx-build; \
  rm -rf /tmp/* /var/tmp/*

STOPSIGNAL SIGQUIT

CMD ["nginx"]
