FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-php:7.2" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG PHP_VERSION=7.2.0alpha1
ARG PHP_CONFIG=/etc/php

ARG REALPATH_TURBO_TAG=v2.0.0rc1
ARG XDEBUG_TAG=XDEBUG_2_5_4
ARG MSGPACK_TAG=msgpack-2.0.2
ARG REDIS_TAG=3.1.2
ARG PHPIREDIS_TAG=v1.0.0
ARG AMQP_BRANCH=release/1.9
ARG PHPV8_TAG=v0.1.6

# https://github.com/AlexMasterov/dockerfiles#alpine-libv8
ARG V8_SOURCE=https://www.dropbox.com/s/80garhhan7js8ef/alpine-libv8-6.1.147.tar.gz
ARG V8_DIR=/usr/local/v8

RUN \
  apk add --update \
    ssmtp \
    tini \

  && addgroup -g 82 -S www-data \
  && adduser -u 82 -S -D -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN \
  apk add --virtual .php-build-dependencies \
    autoconf \
    binutils \
    bzip2-dev \
    curl-dev \
    file \
    freetype-dev \
    g++ \
    gcc \
    git \
    icu-dev \
    jpeg-dev \
    libevent-dev \
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
  && apk add --virtual .php-runtime-dependencies \
    curl \
    freetype \
    icu \
    jpeg \
    libbz2 \
    libevent \
    libltdl \
    libmcrypt \
    libpng \
    libpq \
    libwebp \
    libxml2 \
    libxslt \
    pcre \
    readline \
    zlib \

  # Proper iconv
  && apk add --no-cache --virtual .iconv-runtime-dependencies \
    --repository https://dl-3.alpinelinux.org/alpine/edge/testing/ \
    gnu-libiconv-dev \

  # Replace binary and headers
  && (mv /usr/bin/gnu-iconv /usr/bin/iconv; \
      mv /usr/include/gnu-libiconv/*.h /usr/include; rm -rf /usr/include/gnu-libiconv) \

  # See: https://github.com/docker-library/php/issues/272
  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  && export CFLAGS="-O2 -march=native -fstack-protector-strong -fpic -fpie" \
  && export CPPFLAGS=${CFLAGS} \
  # Use multi-threaded 'make' by default
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \

  # PHP
  && PHP_SOURCE="https://downloads.php.net/~pollita/php-${PHP_VERSION}.tar.xz" \
  && curl -fSL --connect-timeout 30 ${PHP_SOURCE} | tar xJ -C /tmp \
  && cd /tmp/php-${PHP_VERSION} \

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

  # See: https://bugs.php.net/bug.php?id=52312
  && git clone -o ${REALPATH_TURBO_TAG} --depth 1 https://github.com/Whissi/realpath_turbo.git /tmp/realpath_turbo \
  && cd /tmp/realpath_turbo \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # xDebug
  && git clone -o ${XDEBUG_TAG} --depth 1 https://github.com/xdebug/xdebug.git /tmp/xdebug \
  && cd /tmp/xdebug \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Msgpack
  && MSGPACK_TAG="msgpack-2.0.2" \
  && git clone -o ${MSGPACK_TAG} --depth 1 https://github.com/msgpack/msgpack-php.git /tmp/msgpack-php \
  && cd /tmp/msgpack-php \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Redis
  && git clone -o ${REDIS_TAG} --depth 1 https://github.com/phpredis/phpredis.git /tmp/redis \
  && cd /tmp/redis \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Phpiredis
  # https://blog.remirepo.net/post/2016/11/13/Redis-from-PHP
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

  # php-amqp (RabbitMQ)
  && apk add --virtual .amqp-build-dependencies \
    rabbitmq-c-dev \
  && apk add --virtual .amqp-runtime-dependencies \
    rabbitmq-c \

  && git clone -b ${AMQP_BRANCH} --depth 1 https://github.com/pdezwart/php-amqp.git /tmp/php-amqp \
  && cd /tmp/php-amqp \
  && phpize \
  && ./configure \
  && make \
  && make install \

  && apk del .amqp-build-dependencies \

  # libv8
  && (mkdir -p ${V8_DIR}; \
      curl -fSL --connect-timeout 30 ${V8_SOURCE} | tar xz -C ${V8_DIR}) \

  # php-v8
  && git clone -o ${PHPV8_TAG} --depth 1 https://github.com/pinepain/php-v8.git /tmp/php-v8 \
  && cd /tmp/php-v8 \
  && phpize \
  && ./configure \
    --with-v8=${V8_DIR} \
  && make \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .php-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
