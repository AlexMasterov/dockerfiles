FROM alexmasterov/alpine-libv8:6.7 as libv8
FROM alpine:3.9

LABEL repository.hub="alexmasterov/alpine-php:7.1" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG PHP_VERSION=7.1.30
ARG PHP_CONFIG=/etc/php

ARG REALPATH_TURBO_TAG=v2.0.0
ARG XDEBUG_TAG=2.6.1
ARG MSGPACK_TAG=msgpack-2.0.2
ARG REDIS_TAG=4.2.0
ARG MEMCACHED_TAG=v3.1.3
ARG TARANTOOL_VERSION=0.3.2
ARG MONGODB_VERSION=1.5.3
ARG AMQP_TAG=v1.9.4
ARG PHPV8_VERSION=0.2.2

COPY --from=libv8 /usr/local/v8 /usr/local/v8

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
    bison \
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
  && apk add --no-cache --virtual .ext-runtime-dependencies \
    --repository https://nl.alpinelinux.org/alpine/edge/testing/ \
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
    --enable-option-checking=fatal \
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
  && MONGODB_FILENAME="mongodb-${MONGODB_VERSION}" \
  && MONGODB_SOURCE="https://github.com/mongodb/mongo-php-driver/releases/download/${MONGODB_VERSION}/${MONGODB_FILENAME}.tgz" \
  && curl -fSL --connect-timeout 30 ${MONGODB_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${MONGODB_FILENAME} \
  && phpize \
  && ./configure \
    --with-mongodb-ssl=openssl \
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
  && : "---------- php-v8 ----------" \
  && PHPV8_FILENAME="php-v8-${PHPV8_VERSION}" \
  && PHPV8_SOURCE="https://github.com/pinepain/php-v8/archive/v${PHPV8_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${PHPV8_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${PHPV8_FILENAME} \
  && phpize \
  && ./configure \
    --with-v8=/usr/local/v8 \
  && make \
  && make install \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .php-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
