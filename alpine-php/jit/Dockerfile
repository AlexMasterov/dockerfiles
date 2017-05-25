FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-php:jit" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG PHP_CONFIG=/etc/php

RUN \
  apk add --update \
    ssmtp \
    tini \

  && addgroup -g 82 -S www-data \
  && adduser -u 82 -S -D -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN \
  apk add --no-cache --virtual .php-build-dependencies \
    autoconf \
    binutils \
    bison \
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

  # See: https://github.com/docker-library/php/issues/272
  && export CFLAGS="-O2 -march=native -fstack-protector-strong -fpic -fpie" \
  && export CPPFLAGS=${CFLAGS} \
  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  # Use multi-threaded 'make' by default
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \

  # Proper iconv
  && LIBICONV_VERSION="1.15" \
  && LIBICONV_SOURCE="https://ftp.gnu.org/pub/gnu/libiconv/libiconv-${LIBICONV_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${LIBICONV_SOURCE} | tar xz -C /tmp \
  && cd /tmp/libiconv-${LIBICONV_VERSION} \

  && ./configure \
    --prefix=/usr \
  && make \
  && make install \

  # PHP
  && PHP_BRANCH="jit-dynasm" \
  && git clone -b ${PHP_BRANCH} --depth 1 https://github.com/zendtech/php-src.git /tmp/php-src \
  && cd /tmp/php-src \

  && ./buildconf --force \
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
  && REALPATH_TURBO_TAG="v2.0.0rc1" \
  && git clone -o ${REALPATH_TURBO_TAG} --depth 1 https://github.com/Whissi/realpath_turbo.git /tmp/realpath_turbo \
  && cd /tmp/realpath_turbo \
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

  # libevent
  && LIBEVENT_BRANCH="master" \
  && git clone -b ${LIBEVENT_BRANCH} --depth 1 https://github.com/expressif/pecl-event-libevent.git /tmp/libevent \
  && cd /tmp/libevent \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Redis
  && REDIS_TAG="3.1.2" \
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

  && PHPIREDIS_TAG="v1.0.0" \
  && git clone -o ${PHPIREDIS_TAG} --depth 1 https://github.com/nrk/phpiredis.git /tmp/phpiredis \
  && cd /tmp/phpiredis \
  && phpize \
  && ./configure \
  && make \
  && make install \

  && apk del .phpiredis-build-dependencies \

  # Memcached
  && apk add --virtual .memcached-build-dependencies \
    libmemcached-dev \
    cyrus-sasl-dev \
  && apk add --virtual .memcached-runtime-dependencies \
    libmemcached \

  && MEMCACHED_TAG="v3.0.3" \
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

  # Mongodb
  && apk add --virtual .mongodb-build-dependencies \
    automake \
  && apk add --virtual .mongodb-runtime-dependencies \
    libressl2.5-libtls \

  && LIBBSON_VERSION="1.6.3" \
  && LIBBSON_FILENAME="libbson-${LIBBSON_VERSION}" \
  && LIBBSON_SOURCE="https://github.com/mongodb/libbson/releases/download/${LIBBSON_VERSION}/${LIBBSON_FILENAME}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${LIBBSON_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${LIBBSON_FILENAME} \
  && (aclocal; automake) \
  && ./configure \
    --enable-lto \
  && make \
  && make install \

  && LIBMONGOC_VERSION="1.6.3" \
  && LIBMONGOC_FILENAME="mongo-c-driver-${LIBBSON_VERSION}" \
  && LIBMONGOC_SOURCE="https://github.com/mongodb/mongo-c-driver/releases/download/${LIBMONGOC_VERSION}/${LIBMONGOC_FILENAME}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${LIBMONGOC_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${LIBMONGOC_FILENAME} \
  && ./configure \
    --disable-automatic-init-and-cleanup \
    --enable-rdtscp \
    --enable-ssl=libressl \
    --enable-sasl=no \
    --with-libbson=system \
  && make \
  && make install \

  && MONGODB_VERSION="1.2.9" \
  && MONGODB_FILENAME="mongodb-${MONGODB_VERSION}" \
  && MONGODB_SOURCE="https://github.com/mongodb/mongo-php-driver/releases/download/${MONGODB_VERSION}/${MONGODB_FILENAME}.tgz" \
  && curl -fSL --connect-timeout 30 ${MONGODB_SOURCE} | tar xz -C /tmp \
  && cd /tmp/${MONGODB_FILENAME} \
  && phpize \
  && ./configure \
    --with-libbson=system \
    --with-libmongoc=system \
  && make \
  && make install \

  && apk del .mongodb-build-dependencies \

  # php-amqp (RabbitMQ)
  && apk add --virtual .amqp-build-dependencies \
    rabbitmq-c-dev \
  && apk add --virtual .amqp-runtime-dependencies \
    rabbitmq-c \

  && AMQP_TAG="v1.9.0" \
  && git clone -o ${AMQP_TAG} --depth 1 https://github.com/pdezwart/php-amqp.git /tmp/php-amqp \
  && cd /tmp/php-amqp \
  && phpize \
  && ./configure \
  && make \
  && make install \

  && apk del .amqp-build-dependencies \

  # https://github.com/AlexMasterov/dockerfiles#alpine-libv8
  && V8_SOURCE="https://www.dropbox.com/s/gekgnedrvs13k3l/alpine-libv8-6.0.318.tar.gz" \
  && V8_DIR="/usr/local/v8" \
  && mkdir -p ${V8_DIR} \
  && curl -fSL --connect-timeout 30 ${V8_SOURCE} | tar xz -C ${V8_DIR} \

  # php-v8
  && PHPV8_TAG="v0.1.6" \
  && git clone -o ${PHPV8_TAG} --depth 1 https://github.com/pinepain/php-v8.git /tmp/php-v8 \
  && cd /tmp/php-v8 \
  && phpize \
  && ./configure \
    --with-v8=${V8_DIR} \
  && make \
  && make install \

  # graphql-parser
  && apk add --virtual .graphql-build-dependencies \
    bison \
    cmake \
    flex \

  && GRAPHQL_BRANCH="master" \
  && git clone -b ${GRAPHQL_BRANCH} --depth 1 https://github.com/dosten/graphql-parser-php.git /tmp/graphql \

  # libgraphqlparser
  && cd /tmp/graphql/deps/libgraphqlparser \
  && cmake . \
  && make \
  && make install \

  # graphql-parser-php
  && cd /tmp/graphql \
  && phpize \
  && ./configure \
  && make \
  && make install \

  && apk del .graphql-build-dependencies \

  # Removing build dependencies, clean temporary files
  && apk del .php-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
