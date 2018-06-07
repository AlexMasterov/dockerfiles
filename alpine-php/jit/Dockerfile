FROM alexmasterov/alpine-libv8:6.7 as libv8
FROM alpine:3.7

LABEL repository.hub="alexmasterov/alpine-php:jit" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG PHP_CONFIG=/etc/php

ARG REALPATH_TURBO_TAG=v2.0.0
ARG REDIS_TAG=4.0.2
ARG PHPIREDIS_TAG=v1.0.0
ARG MONGODB_VERSION=1.4.4
ARG PHPV8_VERSION=0.2.2
ARG GRAPHQL_BRANCH=master

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
    libzip-dev \
    make \
    pcre2-dev \
    postgresql-dev \
    re2c \
    readline-dev \
    sqlite-dev \
  && apk add --no-cache --virtual .php-build-dependencies \
    --repository https://dl-3.alpinelinux.org/alpine/edge/main/ \
    argon2-dev \
  && : "---------- Proper iconv ----------" \
  && apk add --no-cache --virtual .iconv-runtime-dependencies \
    --repository https://dl-3.alpinelinux.org/alpine/edge/testing/ \
    gnu-libiconv-dev \
  && : "---------- Replace binary and headers ----------" \
  && (mv /usr/bin/gnu-iconv /usr/bin/iconv; \
      mv /usr/include/gnu-libiconv/*.h /usr/include; rm -rf /usr/include/gnu-libiconv) \
  && : "---------- Proper libpcre2 ----------" \
  && (cd /usr/lib; \
      ln -sf libpcre2-posix.a libpcre2.a; ln -sf libpcre2-posix.so libpcre2.so) \
  && : "---------- Build flags ----------" \
  && export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie" \
  && export CFLAGS="-O2 -march=native -fstack-protector-strong -fpic -fpie" \
  && export CPPFLAGS=${CFLAGS} \
  && export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)" \
  && : "---------- PHP ----------" \
  && PHP_BRANCH="jit-dynasm" \
  && git clone -b ${PHP_BRANCH} --depth 1 https://github.com/zendtech/php-src.git /tmp/php-src \
  && cd /tmp/php-src \
  && : "---------- Build ----------" \
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
    --with-mhash \
    --with-openssl=/usr \
      --with-system-ciphers \
    --with-password-argon2 \
    --with-pcre-regex \
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
  && : "---------- MongoDB ----------" \
  && apk add --virtual .mongodb-build-dependencies \
    cmake \
    pkgconfig \
  && apk add --virtual .mongodb-runtime-dependencies \
    libressl2.6-libtls \
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
