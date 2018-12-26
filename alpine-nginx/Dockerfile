FROM alpine:3.8

LABEL repository.hub="alexmasterov/alpine-nginx" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG NGINX_VERSION=1.15.8
ARG NGINX_CONFIG=/etc/nginx

ARG HEADERS_MORE_TAG=v0.33

RUN set -x \
  && apk add --update \
    tini \
  && addgroup -g 82 -S nginx \
  && adduser -u 82 -S -D -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN set -x \
  && apk add --virtual .nginx-build-dependencies \
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
  && : "---------- Build flags ----------" \
  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  && export CFLAGS="-O2 -march=native -flto -fstack-protector-strong -fpic -fpie" \
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \
  && : "---------- Headers More ----------" \
  && HEADERS_MORE_GIT="https://github.com/openresty/headers-more-nginx-module.git" \
  && HEADERS_MORE_DIR="/tmp/headers-more-nginx-module" \
  && git clone -o ${HEADERS_MORE_TAG} --depth 1 ${HEADERS_MORE_GIT} ${HEADERS_MORE_DIR} \
  && : "---------- Nginx ----------" \
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
