FROM alpine:3.4

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-php-dev" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG PHP_VERSION="7.0.13"
ARG PHP_INI_DIR="/etc/php"

ENV LD_PRELOAD /usr/local/lib/preloadable_libiconv.so

RUN \
  addgroup -S www-data \
  && adduser -D -S -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN \
  apk add --update --virtual .php-build-dependencies \
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
    openssl-dev \
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
    openssl \
    readline \
    zlib \

  # Proper iconv
  && LIBICONV_VERSION="1.14" \
  && LIBICONV_SOURCE="https://ftp.gnu.org/pub/gnu/libiconv/libiconv-${LIBICONV_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${LIBICONV_SOURCE} | tar xz -C /tmp \
  && cd /tmp/libiconv-${LIBICONV_VERSION} \

  # Patches
  && LIBICONV_FIX_PATCH="https://git.io/vP8wu" \
  && curl -fSL --connect-timeout 30 ${LIBICONV_FIX_PATCH} | patch -p1 -u \

  && ./configure \
    --prefix=/usr/local \
  && make \
  && make install \

  && libtool --finish /usr/local/lib \
  && rm /usr/bin/iconv \

  # PHP
  && PHP_SOURCE="https://secure.php.net/get/php-${PHP_VERSION}.tar.gz/from/this/mirror" \
  && curl -fSL --connect-timeout 30 ${PHP_SOURCE} | tar xz -C /tmp \
  && cd /tmp/php-${PHP_VERSION} \

  && ./configure \
    --prefix=/usr \
    --sysconfdir=${PHP_INI_DIR} \
    --with-config-file-path=${PHP_INI_DIR} \
    --with-config-file-scan-dir=${PHP_INI_DIR}/conf.d \
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
    --enable-zlib \
    --with-bz2=/usr \
    --with-curl \
    --with-gd \
      --with-freetype-dir=/usr \
      --with-jpeg-dir=/usr \
      --with-png-dir=/usr \
      --with-webp-dir=/usr \
      --with-zlib-dir=/usr \
    --with-iconv \
    --with-mcrypt \
    --with-mhash \
    --with-openssl \
    --with-pcre-regex \
      --with-pcre-jit \
    --with-pdo-mysql=mysqlnd \
    --with-pdo-pgsql \
    --with-pdo-sqlite \
    --with-pgsql \
    --with-readline \
    --with-xmlrpc \
    --with-xsl \
    CFLAGS="-O2 -march=native" \
  && make -j "$(getconf _NPROCESSORS_ONLN)" \
  && make install \

  # See: https://bugs.php.net/bug.php?id=52312
  && REALPATH_TURBO_BRANCH="master" \
  && git clone -b ${REALPATH_TURBO_BRANCH} --depth 1 https://github.com/Mikk3lRo/realpath_turbo.git /tmp/realpath_turbo \
  && cd /tmp/realpath_turbo \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # xDebug
  && XDEBUG_BRANCH="master" \
  && git clone -b ${XDEBUG_BRANCH} --depth 1 https://github.com/xdebug/xdebug.git /tmp/xdebug \
  && cd /tmp/xdebug \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Msgpack
  && MSGPACK_BRANCH="master" \
  && git clone -b ${MSGPACK_BRANCH} --depth 1 https://github.com/msgpack/msgpack-php.git /tmp/msgpack-php \
  && cd /tmp/msgpack-php \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Redis
  && REDIS_BRANCH="php7" \
  && git clone -b ${REDIS_BRANCH} --depth 1 https://github.com/phpredis/phpredis.git /tmp/redis \
  && cd /tmp/redis \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Memcached
  && apk add --virtual .memcached-build-dependencies \
    libmemcached-dev \
    cyrus-sasl-dev \
  && apk add --virtual .memcached-runtime-dependencies \
    libmemcached \

  && MEMCACHED_BRANCH="php7" \
  && git clone -b ${MEMCACHED_BRANCH} --depth 1 https://github.com/php-memcached-dev/php-memcached.git /tmp/memcached-php \
  && cd /tmp/memcached-php \
  && phpize \
  && ./configure \
    --enable-memcached-json \
    --enable-memcached-msgpack \
    --disable-memcached-sasl \
  && make \
  && make install \

  && apk del .memcached-build-dependencies \

  # Tarantool
  && apk add --virtual .tarantool-runtime-dependencies \
    libltdl \

  && TARANTOOL_BRANCH="php7-v2" \
  && git clone -b ${TARANTOOL_BRANCH} --depth 1 https://github.com/tarantool/tarantool-php.git /tmp/tarantool-php \
  && cd /tmp/tarantool-php \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Mongo
  && MONGO_TAG="1.1.9" \
  && git clone -o ${MONGO_TAG} --depth 1 https://github.com/mongodb/mongo-php-driver.git /tmp/mongo-php \
  && git -C /tmp/mongo-php submodule update --init --recursive \
  && cd /tmp/mongo-php \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # libevent
  && LIBEVENT_BRANCH="master" \
  && git clone -b ${LIBEVENT_BRANCH} --depth 1 https://github.com/expressif/pecl-event-libevent.git /tmp/libevent-php \
  && cd /tmp/libevent-php \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # https://github.com/AlexMasterov/dockerfiles/blob/master/alpine-v8-lib/Dockerfile
  && V8_VERSION="5.5-lkgr" \
  && V8_LIB_SOURCE="https://www.dropbox.com/s/1hn1ak0iebuv9m4/alpine-v8-lib_${V8_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${V8_LIB_SOURCE} | tar xz -C / \

  # v8js
  && V8JS_BRANCH="php7" \
  && git clone -b ${V8JS_BRANCH} --depth 1 https://github.com/preillyme/v8js.git /tmp/v8js \
  && cd /tmp/v8js \
  && phpize \
  && ./configure \
    --with-v8js=/usr/local/v8 \
  && make \
  # && make test \
  && make install \

  # php-v8
  && PHPV8_BRANCH="master" \
  && git clone -b ${PHPV8_BRANCH} --depth 1 https://github.com/pinepain/php-v8.git /tmp/php-v8 \
  && cd /tmp/php-v8 \
  && phpize \
  && ./configure \
    --with-v8=/usr/local/v8 \
  && make \
  # && make test \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .php-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ARG MAILHOG_VERSION="v0.2.1"
ADD https://github.com/mailhog/MailHog/releases/download/${MAILHOG_VERSION}/MailHog_linux_amd64 /sbin/mailhog
RUN chmod +x /sbin/mailhog

ARG TINI_VERSION="v0.13.0"
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
