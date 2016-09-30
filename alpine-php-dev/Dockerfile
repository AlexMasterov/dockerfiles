FROM alpine:3.4
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

ENV PHP_VERSION="7.0.11" \
    PHP_INI_DIR="/etc/php"

RUN \
  addgroup -S www-data \
  && adduser -D -S -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN \
  apk add --no-cache --update --virtual .php-build-dependencies \
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
    libwebp-dev \
    libxml2-dev \
    make \
    openssl-dev \
    postgresql-dev \
    re2c \
    readline-dev \
    sqlite-dev \
  && apk add --no-cache --virtual .php-runtime-dependencies \
    curl \
    freetype \
    icu \
    jpeg \
    libbz2 \
    libevent \
    libmcrypt \
    libpng \
    libpq \
    libwebp \
    libxml2 \
    openssl \
    readline \
    tar \
    zlib \

  && PHP_FILENAME="php-${PHP_VERSION}.tar.gz" \
  && PHP_SOURCE="http://php.net/get/${PHP_FILENAME}/from/this/mirror" \

  && curl -fSL ${PHP_SOURCE} -o /tmp/${PHP_FILENAME} \
  && tar xzf /tmp/${PHP_FILENAME} -C /tmp \
  && cd /tmp/php-${PHP_VERSION} \

  && ./configure \
    --prefix=/usr \
    --sysconfdir=${PHP_INI_DIR} \
    --with-config-file-path=${PHP_INI_DIR} \
    --with-config-file-scan-dir=${PHP_INI_DIR}/conf.d \
    --without-pear \
    --disable-debug \
    --disable-phpdbg \
    --disable-ipv6 \
    --disable-cgi \
    --disable-rpath \
    --disable-static \
    --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data \
    --enable-opcache \
    --enable-opcache-file \
    --enable-pcntl \
    --enable-shmop \
    --enable-bcmath \
    --enable-mbregex \
    --enable-mbstring \
    --enable-sockets \
    --enable-soap \
    --enable-ftp \
    --enable-zip \
    --enable-exif \
    --enable-calendar \
    --enable-intl \
    --enable-json \
    --enable-phar \
    --enable-dom \
    --enable-session \
    --enable-gd-native-ttf \
    --enable-libxml --with-libxml-dir=/usr \
    --enable-xml \
    --enable-xmlreader \
    --with-xmlrpc \
    --with-mhash \
    --with-iconv \
    --with-pgsql \
    --with-pdo-pgsql \
    --with-pdo-sqlite \
    --with-pdo-mysql=mysqlnd \
    --with-curl \
    --with-openssl \
    --with-mcrypt \
    --with-mhash \
    --with-pcre-regex \
    --with-readline \
    --with-gd \
    --with-jpeg-dir=/usr \
    --with-webp-dir=/usr \
    --with-png-dir=/usr \
    --with-freetype-dir=/usr \
    --with-bz2=/usr \
    --with-zlib --with-zlib-dir=/usr \
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
  && apk add --no-cache --virtual .memcached-build-dependencies \
    libmemcached-dev \
    cyrus-sasl-dev \
  && apk add --no-cache --virtual .memcached-runtime-dependencies \
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
  && apk add --no-cache --virtual .tarantool-runtime-dependencies \
    libltdl \

  && TARANTOOL_BRANCH="php7-v2" \
  && git clone -b ${TARANTOOL_BRANCH} --depth 1 https://github.com/tarantool/tarantool-php.git /tmp/tarantool-php \
  && cd /tmp/tarantool-php \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # Mongo
  && MONGO_TAG="1.1.8" \
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

  # V8 libs
  # https://github.com/AlexMasterov/dockerfiles/blob/master/alpine-v8-lib/Dockerfile
  && V8_LIB_SOURCE="https://www.dropbox.com/s/n9olfc3rf02azug/alpine-v8-lib_5.5.339.tar" \
  && curl -fSL --connect-timeout 30 ${V8_LIB_SOURCE} -o /tmp/v8-lib.tar \
  && tar xzf /tmp/v8-lib.tar -C / \

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

ENV MAILHOG_VERSION v0.2.1
ADD https://github.com/mailhog/MailHog/releases/download/${MAILHOG_VERSION}/MailHog_linux_amd64 /sbin/mailhog
RUN chmod +x /sbin/mailhog

ENV TINI_VERSION v0.10.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
