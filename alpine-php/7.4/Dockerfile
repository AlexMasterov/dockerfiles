FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-php:7.4" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG PHP_VERSION=7.4.4
ARG PHP_CONFIG=/etc/php

ARG XDEBUG_TAG=2.9.4
ARG REDIS_TAG=5.2.1

RUN set -eux; \
  apk add --no-cache \
    ssmtp \
    tzdata; \
  addgroup -g 82 -S www-data; \
  adduser -u 82 -S -D -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN set -eux; \
  apk add --no-cache --virtual .php-build-dependencies \
    argon2-dev \
    autoconf \
    binutils \
    bison \
    bzip2-dev \
    curl-dev \
    file \
    freetype-dev \
    g++ \
    gcc \
    gd-dev \
    git \
    gnu-libiconv-dev \
    icu-dev \
    jpeg-dev \
    libmcrypt-dev \
    libpng-dev \
    libsodium-dev \
    libtool \
    libwebp-dev \
    libxml2-dev \
    libxslt-dev \
    libzip-dev \
    make \
    oniguruma-dev \
    pcre2-dev \
    postgresql-dev \
    re2c \
    readline-dev \
    sqlite-dev; \
  : "---------- Proper iconv: replace binary and headers ----------"; \
  mv /usr/bin/gnu-iconv /usr/bin/iconv; \
  mv /usr/include/gnu-libiconv/*.h /usr/include; \
  rm -rf /usr/include/gnu-libiconv; \
  : "---------- Proper libpcre2 ----------"; \
  (cd /usr/lib; \
    ln -sf libpcre2-posix.a libpcre2.a; \
    ln -sf libpcre2-posix.so libpcre2.so); \
  : "---------- Build flags ----------"; \
  export LDFLAGS="-Wl,-O2 -Wl,--hash-style=both -pie"; \
  export CFLAGS="-O2 -march=native -fstack-protector-strong -fpic -fpie"; \
  export CPPFLAGS=${CFLAGS}; \
  export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)"; \
  : "---------- PHP ----------"; \
  PHP_SOURCE="https://secure.php.net/get/php-${PHP_VERSION}.tar.xz/from/this/mirror"; \
  wget ${PHP_SOURCE} -T 30 -O - | tar xJ -C /tmp; \
  cd /tmp/php-${PHP_VERSION}; \
  : "---------- Build ----------"; \
  ./configure \
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
    --enable-gd \
    --enable-inline-optimization \
    --enable-intl \
    --enable-json \
    --enable-mbregex \
    --enable-mbstring \
    --enable-opcache \
      --enable-huge-code-pages \
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
    --with-sqlite3=/usr \
    --with-bz2=/usr \
    --with-curl=/usr \
    --with-libxml=/usr \
    --with-sodium=/usr \
    --with-external-gd=/usr \
      --with-freetype=/usr \
      --with-jpeg=/usr \
      --with-webp=/usr \
      --with-xpm=no \
    --with-iconv=/usr \
    --with-mhash \
    --with-zip=/usr \
    --with-openssl=/usr \
      --with-system-ciphers \
    --with-password-argon2 \
    --with-pcre-jit \
    --with-pdo-mysql=mysqlnd \
    --with-pdo-pgsql \
    --with-pdo-sqlite=/usr \
    --with-pgsql \
    --with-readline=/usr \
    --with-xmlrpc \
    --with-xsl=/usr; \
  make; \
  make install; \
  runtimeDeps="$( \
    scanelf --needed --nobanner --recursive /usr/sbin/php-fpm \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
	)"; \
  apk add --virtual .php-runtime-dependencies \
    ${runtimeDeps}; \
  : "---------- xDebug ----------"; \
  git clone -o ${XDEBUG_TAG} --depth 1 https://github.com/xdebug/xdebug.git /tmp/xdebug; \
  cd /tmp/xdebug; \
  phpize; \
  ./configure; \
  make; \
  make install; \
  : "---------- Redis ----------"; \
  git clone -o ${REDIS_TAG} --depth 1 https://github.com/phpredis/phpredis.git /tmp/redis; \
  cd /tmp/redis; \
  phpize; \
  ./configure; \
  make; \
  make install; \
  : "---------- Strip files ----------"; \
  find /usr/bin /usr/sbin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "---------- Removing build dependencies, clean temporary files ----------"; \
  apk del --no-network .php-build-dependencies; \
  rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

STOPSIGNAL SIGQUIT

CMD ["php-fpm"]
