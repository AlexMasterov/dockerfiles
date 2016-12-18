FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-php:7.1" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG PHP_VERSION="7.1.0"
ARG PHP_INI_DIR="/etc/php"

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
    --prefix=/usr \
  && make \
  && make install \

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
    --with-bz2=/usr \
    --with-curl \
    --with-gd \
      --with-freetype-dir=/usr \
      --with-jpeg-dir=/usr \
      --with-png-dir=/usr \
      --with-webp-dir=/usr \
      --with-zlib-dir=/usr \
    --with-iconv=/usr \
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
  && REALPATH_TURBO_TAG="v1.4" \
  && git clone -b ${REALPATH_TURBO_TAG} --depth 1 https://github.com/Mikk3lRo/realpath_turbo.git /tmp/realpath_turbo \
  && cd /tmp/realpath_turbo \
  && phpize \
  && ./configure \
  && make \
  && make install \

  # xDebug
  && XDEBUG_TAG="XDEBUG_2_5_0" \
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

  # libevent
  && LIBEVENT_BRANCH="master" \
  && git clone -b ${LIBEVENT_BRANCH} --depth 1 https://github.com/expressif/pecl-event-libevent.git /tmp/libevent \
  && cd /tmp/libevent \
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

  && MEMCACHED_BRANCH="php7" \
  && git clone -b ${MEMCACHED_BRANCH} --depth 1 https://github.com/php-memcached-dev/php-memcached.git /tmp/php-memcached \
  && cd /tmp/php-memcached \
  && phpize \
  && ./configure \
    --disable-memcached-sasl \
    --enable-memcached-msgpack \
    --enable-memcached-json \
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

  # Mongodb
  && apk add --virtual .mongodb-build-dependencies \
    automake \
  && apk add --virtual .mongodb-runtime-dependencies \
    libressl2.4-libtls \

  && LIBBSON_BRANCH="master" \
  && git clone -b ${LIBBSON_BRANCH} --depth 1 https://github.com/mongodb/libbson.git /tmp/libbson \
  && cd /tmp/libbson \
  && ./autogen.sh \
    --enable-lto \
  && make \
  && make install \

  && LIBMONGOC_BRANCH="master" \
  && git clone -b ${LIBMONGOC_BRANCH} --depth 1 https://github.com/mongodb/mongo-c-driver.git /tmp/libmongoc \
  && cd /tmp/libmongoc \
  && ./autogen.sh \
    --disable-automatic-init-and-cleanup \
    --enable-rdtscp \
    --enable-ssl=libressl \
    --enable-sasl=no \
    --with-libbson=system \
  && make \
  && make install \

  && MONGODB_TAG="1.2.2" \
  && git clone -o ${MONGODB_TAG} --depth 1 https://github.com/mongodb/mongo-php-driver.git /tmp/mongo-php \
  && git -C /tmp/mongo-php submodule update \
    --init --recursive --jobs "$(getconf _NPROCESSORS_ONLN)" \
  && cd /tmp/mongo-php \
  && phpize \
  && ./configure \
    --with-libbson=system \
    --with-libmongoc=system \
    STD_CFLAGS="-march=native" \
  && make \
  && make install \

  && apk del .mongodb-build-dependencies \

  # php-amqp (RabbitMQ)
  && apk add --update-cache --virtual .amqp-build-dependencies \
    --repository http://dl-3.alpinelinux.org/alpine/edge/testing/ --allow-untrusted \
    rabbitmq-c-dev \
  && apk add --virtual .rabbitmq-runtime-dependencies \
    rabbitmq-c \

  && AMQP_TAG="v1.7.1" \
  && git clone -o ${AMQP_TAG} --depth 1 https://github.com/pdezwart/php-amqp.git /tmp/php-amqp \
  && cd /tmp/php-amqp \
  && phpize \
  && ./configure \
  && make \
  && make install \

  && apk del .amqp-build-dependencies \

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
  && PHPV8_COMMIT="8680f52" \
  && git clone -b master https://github.com/pinepain/php-v8.git /tmp/php-v8 \
  && git -C /tmp/php-v8 checkout ${PHPV8_COMMIT} \
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

ARG TINI_VERSION="v0.13.2"
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["php-fpm"]
