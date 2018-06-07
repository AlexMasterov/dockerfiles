FROM alpine:3.7

LABEL repository.hub="alexmasterov/alpine-php:7.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG PHP_VERSION=7.0.30
ARG PHP_CONFIG=/etc/php

ARG REALPATH_TURBO_TAG=v2.0.0
ARG XDEBUG_TAG=2.6.0
ARG MSGPACK_TAG=msgpack-2.0.2
ARG REDIS_TAG=4.0.2
ARG PHPIREDIS_TAG=v1.0.0
ARG MEMCACHED_TAG=v3.0.4
ARG TARANTOOL_VERSION=0.3.2
ARG MONGODB_VERSION=1.4.4
ARG AMQP_TAG=v1.9.3
ARG PHPV8_VERSION=0.1.9
ARG GRAPHQL_BRANCH=master

# https://github.com/AlexMasterov/dockerfiles#alpine-libv8
ARG V8_SOURCE=https://www.dropbox.com/s/lhd6x20r47z39v0/alpine-libv8-6.4.34.tar.gz
ARG V8_DIR=/usr/local/v8

RUN set -x \
  && apk add --update \
    ssmtp \
    tini \
  && addgroup -g 82 -S www-data \
  && adduser -u 82 -S -D -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN set -x \
  && apk add --virtual .php-build-dependencies \
    autoconf \
    binutils \
    bzip2-dev \
    curl \
    curl-dev \
    file \
    freetype-dev \
    g++ \
    gcc \
    git \
    icu-dev \
    jpeg-dev \
    libmcrypt-dev \
    libpng-dev \
    libtool \
    libwebp-dev \
    libxml2-dev \
    libxslt-dev \
    make \
    pcre-dev \
    postgresql-dev \
    re2c \
    readline-dev \
    sqlite-dev \
  && : "---------- Proper iconv ----------" \
  && apk add --no-cache --virtual .iconv-runtime-dependencies \
    --repository https://dl-3.alpinelinux.org/alpine/edge/testing/ \
    gnu-libiconv-dev \
  && : "---------- Replace binary and headers ----------" \
  && (mv /usr/bin/gnu-iconv /usr/bin/iconv; \
      mv /usr/include/gnu-libiconv/*.h /usr/include; rm -rf /usr/include/gnu-libiconv) \
  && : "---------- Build flags ----------" \
  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  && export CFLAGS="-O2 -march=native -fstack-protector-strong -fpic -fpie" \
  && export CPPFLAGS=${CFLAGS} \
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \
  && : "---------- PHP ----------" \
  && PHP_SOURCE="https://secure.php.net/get/php-${PHP_VERSION}.tar.xz/from/this/mirror" \
  && curl -fSL --connect-timeout 30 ${PHP_SOURCE} | tar xJ -C /tmp \
  && cd /tmp/php-${PHP_VERSION} \
  && : "---------- Build ----------" \
  && ./configure \
    --prefix=/usr \
    --sysconfdir=${PHP_CONFIG} \
    --with-config-file-path=${PHP_CONFIG} \
    --with-config-file-scan-dir=${PHP_CONFIG}/conf.d \
    --without-pear \
    --disable-cgi \
    --disable-debug \
    --disable-ipv6 \
    --disable-phpdbg \
    --disable-rpath \
    --disable-static \
    --enable-bcmath \
    --enable-calendar \
    --enable-dom \
    --enable-exif \
    --enable-fd-setsize=$(ulimit -n) \
    --enable-fpm \
      --with-fpm-group=www-data \
      --with-fpm-user=www-data \
    --enable-ftp \
    --enable-inline-optimization \
    --enable-intl \
    --enable-json \
    --enable-libxml \
      --with-libxml-dir=/usr \
    --enable-mbregex \
    --enable-mbstring \
    --enable-opcache \
      --enable-huge-code-pages \
      --enable-opcache-file \
    --enable-pcntl \
    --enable-phar \
    --enable-session \
    --enable-shmop \
    --enable-soap \
    --enable-sockets \
    --enable-xml \
    --enable-xmlreader \
    --enable-xmlwriter \
    --enable-zip \
      --with-pcre-dir=/usr \
    --with-bz2=/usr \
    --with-curl=/usr \
    --with-gd \
      --with-freetype-dir=/usr \
      --with-jpeg-dir=/usr \
      --with-png-dir=/usr \
      --with-webp-dir=/usr \
      --with-xpm-dir=no \
      --with-zlib-dir=/usr \
    --with-iconv=/usr \
    --with-mcrypt=/usr \
    --with-mhash \
    --with-openssl=/usr \
      --with-system-ciphers \
    --with-pcre-regex=/usr \
      --with-pcre-jit \
    --with-pdo-mysql=mysqlnd \
    --with-pdo-pgsql \
    --with-pdo-sqlite \
    --with-pgsql \
    --with-readline=/usr \
    --with-xmlrpc \
    --with-xsl=/usr \
  && make \
  && make install \
  && runtimeDeps="$( \
    scanelf --needed --nobanner --recursive /usr/sbin/php-fpm \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
	)" \
  && apk add --virtual .php-runtime-dependencies \
    ${runtimeDeps} \
  && : "---------- https://bugs.php.net/bug.php?id=52312 ----------" \
  && git clone -o ${REALPATH_TURBO_TAG} --depth 1 https://github.com/Whissi/realpath_turbo.git /tmp/realpath_turbo \
  && cd /tmp/realpath_turbo \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && : "---------- xDebug ----------" \
  && git clone -o ${XDEBUG_TAG} --depth 1 https://github.com/xdebug/xdebug.git /tmp/xdebug \
  && cd /tmp/xdebug \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && : "---------- Msgpack ----------" \
  && git clone -o ${MSGPACK_TAG} --depth 1 https://github.com/msgpack/msgpack-php.git /tmp/msgpack-php \
  && cd /tmp/msgpack-php \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && : "---------- Redis ----------" \
  && git clone -o ${REDIS_TAG} --depth 1 https://github.com/phpredis/phpredis.git /tmp/redis \
  && cd /tmp/redis \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && : "---------- Phpiredis ----------" \
  && : "---------- https://blog.remirepo.net/post/2016/11/13/Redis-from-PHP ----------" \
  && apk add --virtual .phpiredis-build-dependencies \
    hiredis-dev \
  && apk add --virtual .phpiredis-runtime-dependencies \
    hiredis \
  && git clone -o ${PHPIREDIS_TAG} --depth 1 https://github.com/nrk/phpiredis.git /tmp/phpiredis \
  && cd /tmp/phpiredis \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && apk del .phpiredis-build-dependencies \
  && : "---------- Memcached ----------" \
  && apk add --virtual .memcached-build-dependencies \
    libmemcached-dev \
    cyrus-sasl-dev \
  && apk add --virtual .memcached-runtime-dependencies \
    libmemcached \
  && git clone -o ${MEMCACHED_TAG} --depth 1 https://github.com/php-memcached-dev/php-memcached.git /tmp/php-memcached \
  && cd /tmp/php-memcached \
  && phpize \
  && ./configure \
    --disable-memcached-sasl \
    --enable-memcached-msgpack \
    --enable-memcached-json \
  && make \
  && make install \
  && apk del .memcached-build-dependencies \
  && : "---------- Tarantool ----------" \
  && apk add --virtual .tarantool-runtime-dependencies \
    libltdl \
  && TARANTOOL_FILENAME="tarantool-php-${TARANTOOL_VERSION}" \
  && TARANTOOL_SOURCE="https://github.com/tarantool/tarantool-php/archive/${TARANTOOL_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${TARANTOOL_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${TARANTOOL_FILENAME} \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && : "---------- Mongodb ----------" \
  && apk add --virtual .mongodb-build-dependencies \
    cmake \
    pkgconfig \
  && apk add --virtual .mongodb-runtime-dependencies \
    libressl2.6-libtls \
  && : "---------- mongo-php-driver ----------" \
  && MONGODB_FILENAME="mongodb-${MONGODB_VERSION}" \
  && MONGODB_SOURCE="https://github.com/mongodb/mongo-php-driver/releases/download/${MONGODB_VERSION}/${MONGODB_FILENAME}.tgz" \
  && curl -fSL --connect-timeout 30 ${MONGODB_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${MONGODB_FILENAME} \
  && phpize \
  && ./configure \
    --with-mongodb-ssl=libressl \
  && make \
  && make install \
  && apk del .mongodb-build-dependencies \
  && : "---------- php-amqp (RabbitMQ) ----------" \
  && apk add --virtual .amqp-build-dependencies \
    rabbitmq-c-dev \
  && apk add --virtual .amqp-runtime-dependencies \
    rabbitmq-c \
  && git clone -o ${AMQP_TAG} --depth 1 https://github.com/pdezwart/php-amqp.git /tmp/php-amqp \
  && cd /tmp/php-amqp \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && apk del .amqp-build-dependencies \
  && : "---------- libv8 ----------" \
  && (mkdir -p ${V8_DIR}; \
      curl -fSL --connect-timeout 30 ${V8_SOURCE} | tar xz -C ${V8_DIR}) \
  && : "---------- php-v8 ----------" \
  && PHPV8_FILENAME="php-v8-${PHPV8_VERSION}" \
  && PHPV8_SOURCE="https://github.com/pinepain/php-v8/archive/v${PHPV8_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${PHPV8_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${PHPV8_FILENAME} \
  && phpize \
  && ./configure \
    --with-v8=${V8_DIR} \
  && make \
  && make install \
  && : "---------- graphql-parser ----------" \
  && apk add --virtual .graphql-build-dependencies \
    bison \
    cmake \
    flex \
    python2 \
  && git clone -b ${GRAPHQL_BRANCH} --depth 1 https://github.com/dosten/graphql-parser-php.git /tmp/graphql \
  && : "---------- libgraphqlparser ----------" \
  && cd /tmp/graphql/deps/libgraphqlparser \
  && cmake . \
  && make \
  && make install \
  && : "---------- graphql-parser-php ----------" \
  && cd /tmp/graphql \
  && phpize \
  && ./configure \
  && make \
  && make install \
  && apk del .graphql-build-dependencies \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .php-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
