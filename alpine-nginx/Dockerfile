FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-nginx" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.19.2
ARG NGINX_CONFIG=/etc/nginx

ARG HEADERS_MORE_TAG=v0.33

RUN set -eux; \
  addgroup -g 82 -S nginx; \
  adduser -u 82 -S -D -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN set -eux; \
  apk add --no-cache --virtual .nginx-build \
    g++ \
    gcc \
    git \
    linux-headers \
    make \
    openssl-dev \
    pcre-dev \
    zlib-dev; \
  : "-- Setup build flags"; \
  export \
    LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
    CFLAGS="-O2 -march=native -flto -fstack-protector-strong -fpic -fpie" \
    MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)"; \
  : "-- Download 'Headers More' module"; \
  HEADERS_MORE_GIT="https://github.com/openresty/headers-more-nginx-module.git"; \
  HEADERS_MORE_DIR="/tmp/headers-more-nginx-module"; \
  git clone -o $HEADERS_MORE_TAG --depth 1 $HEADERS_MORE_GIT $HEADERS_MORE_DIR; \
  : "-- Download Nginx"; \
  NGINX_SOURCE="https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz"; \
  wget $NGINX_SOURCE -T 30 -O - | tar xz -C /tmp; \
  : "-- Build Nginx"; \
  cd /tmp/nginx-$NGINX_VERSION; \
  ./configure \
    --prefix=$NGINX_CONFIG \
    --conf-path=$NGINX_CONFIG/nginx.conf \
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
    --add-module=$HEADERS_MORE_DIR; \
  make; \
  make install; \
  runtimeDeps="$( \
    scanelf --needed --nobanner -F '%n #f' /usr/sbin/nginx /usr/lib/nginx/modules/*.so \
      | tr ',' '\n' \
      | sort -u \
      | sed -e 's/^/so:\1/' \
      | xargs -r apk info --installed \
    )"; \
  apk add --no-cache --virtual .nginx-runtime \
    $runtimeDeps \
    tzdata; \
  : "-- Strip files"; \
  find /usr/sbin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Removing build dependencies, clean temporary files"; \
  apk del .nginx-build; \
  rm -rf /var/tmp/* /tmp/*

STOPSIGNAL SIGTERM

CMD ["nginx"]
