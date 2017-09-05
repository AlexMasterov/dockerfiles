FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-nginx" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.13.5
ARG NGINX_CONFIG=/etc/nginx

ARG HEADERS_MORE_TAG=v0.32

ARG UID=82
ARG GID=82

RUN \
  apk add --update \
    tini \

  && addgroup -g ${GID} -S nginx \
  && adduser -u ${UID} -S -D -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN \
  apk add --virtual .nginx-build-dependencies \
    curl \
    g++ \
    gcc \
    git \
    libressl-dev \
    linux-headers \
    make \
    pcre-dev \
    zlib-dev \
  && apk add --virtual .nginx-runtime-dependencies \
    ca-certificates \
    pcre \

  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  && export CFLAGS="-O2 -march=native -flto -fstack-protector-strong -fpic -fpie" \
  # Use multi-threaded 'make' by default
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \

  # Headers More
  && HEADERS_MORE_GIT="https://github.com/openresty/headers-more-nginx-module.git" \
  && HEADERS_MORE_DIR="/tmp/headers-more-nginx-module" \
  && git clone -o ${HEADERS_MORE_TAG} --depth 1 ${HEADERS_MORE_GIT} ${HEADERS_MORE_DIR} \

  # Nginx
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
    --add-module=${HEADERS_MORE_DIR} \
  && make \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .nginx-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["nginx"]
