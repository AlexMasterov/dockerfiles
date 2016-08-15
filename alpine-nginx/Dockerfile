FROM alpine:3.4
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

ENV NGINX_VERSION 1.11.3
ENV NGINX_INI_DIR /etc/nginx

ADD \
  # See: https://blog.cloudflare.com/the-curious-case-of-slow-downloads/
  https://raw.githubusercontent.com/cloudflare/cloudflare-blog/master/2016-03-slow-downloads/nginx_send_minimum_rate.patch \
  /tmp/nginx-patches/

RUN \
  addgroup -S nginx \
  && adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx

RUN \
  apk add --update --no-cache --virtual .nginx-build-dependencies \
    linux-headers \
    curl \
    g++ \
    gcc \
    git \
    make \
    openssl-dev \
    patch \
    pcre-dev \
    zlib-dev \
  && apk add --no-cache --virtual .nginx-runtime-dependencies \
    ca-certificates \
    pcre \

  # Install "Headers More" module
  && HEADERS_MORE_BRANCH="master" \
  && HEADERS_MORE_GIT="https://github.com/openresty/headers-more-nginx-module.git" \
  && HEADERS_MORE_DIR="/tmp/headers-more-nginx-module" \

  && git clone -b ${HEADERS_MORE_BRANCH} --depth 1 ${HEADERS_MORE_GIT} ${HEADERS_MORE_DIR} \

  # Install Nginx
  && NGINX_FILENAME="nginx-${NGINX_VERSION}.tar.gz" \
  && NGINX_SOURCE="http://nginx.org/download/${NGINX_FILENAME}" \

  && curl -fSL ${NGINX_SOURCE} -o /tmp/${NGINX_FILENAME} \
  && tar xzf /tmp/${NGINX_FILENAME} -C /tmp \
  && cd /tmp/nginx-${NGINX_VERSION} \

  # Applying patches
  && patch -p0 -i /tmp/nginx-patches/* \

  && ./configure \
    --prefix=${NGINX_INI_DIR} \
    --conf-path=${NGINX_INI_DIR}/nginx.conf \
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
    --user=nginx  \
    --group=nginx  \
    --without-http_autoindex_module \
    --without-http_ssi_module \
    --with-http_v2_module \
    --with-http_ssl_module \
    --with-http_dav_module \
    --with-http_slice_module \
    --with-http_realip_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_stub_status_module \
    --with-http_secure_link_module \
    --with-http_auth_request_module \
    --with-http_degradation_module \
    --with-pcre \
    --with-pcre-jit \
    --with-stream \
    --with-stream_ssl_module  \
    --with-threads \
    --with-file-aio \
    --with-cc-opt='-Ofast -march=native -flto -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security' \
    --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,--as-needed' \
    --add-module=${HEADERS_MORE_DIR} \
  && make -j "$(getconf _NPROCESSORS_ONLN)" \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .nginx-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ENV TINI_VERSION v0.10.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["nginx"]
