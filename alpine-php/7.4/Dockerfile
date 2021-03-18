FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-php:7.4" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG PHP_VERSION=7.4.16
ARG PHP_CONFIG=/etc/php

ARG PHP_CFLAGS="-march=native -O2 -fpic -fpie -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ARG PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ARG PHP_ARGON2=YES
ARG PHP_CLI=YES
ARG PHP_CTYPE=YES
ARG PHP_FILEINFO=NO
ARG PHP_FPM=NO
ARG PHP_FTP=NO
ARG PHP_INTL=NO
ARG PHP_IPV6=NO
ARG PHP_PCNTL=NO
ARG PHP_PHAR=NO
ARG PHP_POSIX=NO
ARG PHP_TOKENIZER=YES

ARG EXT_BCMATH=NO
ARG EXT_BZ2=NO
ARG EXT_CURL=YES
ARG EXT_DOM=NO
ARG EXT_GD=YES
ARG EXT_GD_FREETYPE=NO
ARG EXT_GD_JPEG=NO
ARG EXT_GD_WEBP=NO
ARG EXT_GD_XPM=NO
ARG EXT_GMP=NO
ARG EXT_LIBXML=YES
ARG EXT_MBREGEX=NO
ARG EXT_OPENSSL=YES
ARG EXT_OPENSSL_SYSTEM_CIPHERS=YES
ARG EXT_PCRE=EXTERNAL
ARG EXT_PDO=YES
ARG EXT_PDO_MYSQL=NO
ARG EXT_PDO_PGSQL=YES
ARG EXT_PDO_SQLITE=YES
ARG EXT_POSTGRESQL=NO
ARG EXT_SIMPLEXML=YES
ARG EXT_SODIUM=YES
ARG EXT_SQLITE=NO
ARG EXT_XML=NO
ARG EXT_XMLREADER=NO
ARG EXT_XMLWRITER=NO
ARG EXT_XSL=NO
ARG EXT_ZIP=NO

ARG EXT_XDEBUG=NO
ARG EXT_XDEBUG_VERSION=3.0.3

ARG EXT_APCU=NO
ARG EXT_APCU_VERSION=5.1.20
ARG EXT_APCU_RWLOCKS=YES
ARG EXT_APCU_SPINLOCKS=NO

ARG EXT_MSGPACK=NO
ARG EXT_MSGPACK_VERSION=2.1.1

ARG EXT_PHPREDIS=NO
ARG EXT_PHPREDIS_VERSION=5.3.3
ARG EXT_PHPREDIS_LZ4=NO
ARG EXT_PHPREDIS_ZSTD=NO
ARG EXT_PHPREDIS_SESSION=YES
ARG EXT_PHPREDIS_JSON=YES
ARG EXT_PHPREDIS_MSGPACK=NO

ARG EXT_MEMCACHED=NO
ARG EXT_MEMCACHED_VERSION=3.1.5
ARG EXT_MEMCACHED_SESSION=YES
ARG EXT_MEMCACHED_JSON=YES
ARG EXT_MEMCACHED_MSGPACK=NO

ARG EXT_MONGODB=NO
ARG EXT_MONGODB_VERSION=1.9.0
ARG EXT_MONGODB_SSL=OPENSSL

ARG EXT_AMQP=NO
ARG EXT_AMQP_VERSION=1.11.0beta

RUN set -eux; \
  addgroup -g 82 -S www-data; \
  adduser -u 82 -S -D -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN set -eux; \
  if_eq() { [ ${1} = ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xz} -C /tmp; }; \
  make_ext() { phpize; ./configure ${*:-}; make; make install; }; \
  apk add --no-cache --virtual .php-build \
    autoconf \
    bison \
    file \
    g++ \
    gcc \
    gnu-libiconv-dev \
    make \
    re2c \
    zlib-dev \
    $(if_eq ${PHP_INTL}       'YES'      'icu-dev') \
    $(if_eq ${PHP_ARGON2}     'YES'      'argon2-dev') \
    $(if_eq ${PHP_READLINE}   'YES'      'readline-dev') \
    $(if_eq ${EXT_ZIP}        'YES'      'libzip-dev') \
    $(if_eq ${EXT_SODIUM}     'YES'      'libsodium-dev') \
    $(if_eq ${EXT_GMP}        'YES'      'gmp-dev') \
    $(if_eq ${EXT_CURL}       'YES'      'curl-dev') \
    $(if_eq ${EXT_LIBXML}     'YES'      'libxml2-dev') \
    $(if_eq ${EXT_BZ2}        'YES'      'bzip2-dev') \
    $(if_eq ${EXT_SQLITE}     'YES'      'sqlite-dev') \
    $(if_eq ${EXT_POSTGRESQL} 'YES'      'postgresql-dev') \
    $(if_eq ${EXT_OPENSSL}    'YES'      'openssl-dev') \
    $(if_eq ${EXT_OPENSSL}    'LIBRESSL' 'libressl-dev') \
    $(if_eq ${EXT_PCRE}       'EXTERNAL' 'pcre2-dev') \
    $(if_eq ${EXT_MBREGEX}    'YES'      'oniguruma-dev') \
    $([ ${EXT_PDO} = 'YES' ] && echo \
      $(if_eq ${EXT_PDO_SQLITE} 'YES'     'sqlite-dev') \
      $(if_eq ${EXT_PDO_PGSQL}  'YES'     'postgresql-dev') \
      $(if_eq ${EXT_PDO_MYSQL}  'MARIADB' 'mariadb-dev') \
    ) \ 
    $([ ${EXT_GD} = 'YES' ] \
      && echo \
        libpng-dev \
        $(if_eq ${EXT_GD_JPEG}     'EXTERNAL' 'jpeg-dev') \
        $(if_eq ${EXT_GD_XPM}      'EXTERNAL' 'libxpm-dev') \
        $(if_eq ${EXT_GD_WEBP}     'EXTERNAL' 'libwebp-dev') \
        $(if_eq ${EXT_GD_FREETYPE} 'EXTERNAL' 'freetype-dev') \
      || echo \
        $(if_eq ${EXT_GD}          'EXTERNAL' 'gd-dev') \
    ) \
    $(if_eq ${EXT_PHPREDIS_LZ4}  'YES'      'lz4-dev') \
    $(if_eq ${EXT_PHPREDIS_ZSTD} 'YES'      'zstd-dev') \
    $(if_eq ${EXT_MEMCACHED}     'YES'      'libmemcached-dev') \
    $(if_eq ${EXT_MONGODB_SSL}   'OPENSSL'  'openssl-dev') \
    $(if_eq ${EXT_MONGODB_SSL}   'LIBRESSL' 'libressl-dev') \
    $(if_eq ${EXT_AMQP}          'YES'      'rabbitmq-c-dev'); \ 
  : "-- Proper iconv: replace binary and headers"; \
  mv /usr/bin/gnu-iconv /usr/bin/iconv; \
  mv /usr/include/gnu-libiconv/*.h /usr/include; \
  rm -rf /usr/include/gnu-libiconv; \
  : "-- Download PHP source"; \
  load_archive "https://secure.php.net/get/php-${PHP_VERSION}.tar.xz/from/this/mirror" xJ; \
  cd /tmp/php-${PHP_VERSION}; \
  : "-- Setup build flags"; \
  export MAKEFLAGS="-j $(expr $(getconf _NPROCESSORS_ONLN) \+ 1)"; \
  : "-- Build PHP"; \
  ./configure \
    --prefix=/usr \
    --sysconfdir=${PHP_CONFIG} \
    --with-config-file-path=${PHP_CONFIG} \
    --with-config-file-scan-dir="${PHP_CONFIG}/conf.d" \
    --with-layout="GNU" \
    --without-pear \
    --disable-cgi \
    --disable-debug \
    --disable-phpdbg \
    --disable-rpath \
    --enable-option-checking=fatal \
    --enable-fd-setsize=$(ulimit -n) \
    --enable-mbstring \
    --with-mhash \
    --with-zlib=/usr \
    --with-iconv=/usr \
    $(if_eq ${PHP_CLI}       'NO'  '--disable-cli') \
    $(if_eq ${PHP_IPV6}      'NO'  '--disable-ipv6') \
    $(if_eq ${PHP_POSIX}     'NO'  '--disable-posix') \
    $(if_eq ${PHP_CTYPE}     'NO'  '--disable-ctype') \
    $(if_eq ${PHP_PHAR}      'NO'  '--disable-phar') \
    $(if_eq ${PHP_FILEINFO}  'NO'  '--disable-fileinfo') \
    $(if_eq ${PHP_TOKENIZER} 'NO'  '--disable-tokenizer') \
    $(if_eq ${EXT_MBREGEX}   'NO'  '--disable-mbregex') \
    $(if_eq ${PHP_FPM}       'YES' '--enable-fpm') \
    $(if_eq ${PHP_INTL}      'YES' '--enable-intl') \
    $(if_eq ${PHP_PCNTL}     'YES' '--enable-pcntl') \
    $(if_eq ${PHP_FTP}       'YES' '--enable-ftp') \
    $(if_eq ${PHP_ARGON2}    'YES' '--with-password-argon2') \
    $(if_eq ${EXT_BCMATH}    'YES' '--enable-bcmath') \
    $(if_eq ${EXT_SODIUM}    'YES' '--with-sodium=/usr') \
    $(if_eq ${EXT_CURL}      'YES' '--with-curl=/usr') \
    $(if_eq ${EXT_ZIP}       'YES' '--with-zip=/usr') \
    $(if_eq ${EXT_GMP}       'YES' '--with-gmp=/usr') \
    $(if_eq ${EXT_BZ2}       'YES' '--with-bz2=/usr') \
    $(if_eq ${EXT_SQLITE}    'YES' '--with-sqlite3=/usr' '--without-sqlite3') \
    $([ ${EXT_OPENSSL} != 'NO' ] && echo \
      --with-openssl=/usr \
      $(if_eq ${EXT_OPENSSL_SYSTEM_CIPHERS} 'YES' '--with-system-ciphers') \
    \ ) \
    $([ ${EXT_PDO} = 'YES' ] \
      && echo \
        $(if_eq ${EXT_PDO_SQLITE} 'YES'     '--with-pdo-sqlite=/usr' '--without-pdo-sqlite') \
        $(if_eq ${EXT_PDO_PGSQL}  'YES'     '--with-pdo-pgsql=/usr') \
        $(if_eq ${EXT_PDO_MYSQL}  'YES'     '--with-pdo-mysql=mysqlnd') \
        $(if_eq ${EXT_PDO_MYSQL}  'MARIADB' '--with-pdo-mysql=/usr') \
      || echo \
        --disable-pdo \
    ) \
    $([ ${EXT_LIBXML} = 'YES' ] \
      && echo \
        $(if_eq ${EXT_XML}       'NO'  '--disable-xml') \
        $(if_eq ${EXT_XMLREADER} 'NO'  '--disable-xmlreader') \
        $(if_eq ${EXT_XMLWRITER} 'NO'  '--disable-xmlwriter') \
        $(if_eq ${EXT_SIMPLEXML} 'NO'  '--disable-simplexml') \
        $(if_eq ${EXT_DOM}       'NO'  '--disable-dom') \
        $(if_eq ${EXT_XSL}       'YES' '--with-xsl=/usr') \
      || echo \
        --without-libxml \
        --disable-xml \
        --disable-xmlreader \
        --disable-xmlwriter \
        --disable-simplexml \
        --disable-dom \
    ) \
    $(if_eq ${EXT_GD}   'YES'      '--enable-gd') \
    $(if_eq ${EXT_GD}   'EXTERNAL' '--enable-gd --with-external-gd=/usr') \
    $(if_eq ${EXT_PCRE} 'EXTERNAL' '--with-external-pcre=/usr') \
    CFLAGS=${PHP_CFLAGS} \
    CPPFLAGS=${PHP_CFLAGS} \
    LDFLAGS=${PHP_LDFLAGS}; \
  make; \
  make install; \
  if [ ${EXT_XDEBUG} = 'YES' ]; then \
    load_archive "https://github.com/xdebug/xdebug/archive/${EXT_XDEBUG_VERSION}.tar.gz"; \
    cd /tmp/xdebug-${EXT_XDEBUG_VERSION}; \
    make_ext; \
  fi; \
  if [ ${EXT_APCU} = 'YES' ]; then \
    load_archive "https://github.com/krakjoe/apcu/archive/v${EXT_APCU_VERSION}.tar.gz"; \
    cd /tmp/apcu-${EXT_APCU_VERSION}; \
    make_ext \
      --$(if_eq ${EXT_APCU_RWLOCKS}   'YES' 'enable' 'disable')-apcu-rwlocks \
      --$(if_eq ${EXT_APCU_SPINLOCKS} 'YES' 'enable' 'disable')-apcu-spinlocks; \
  fi; \
  if [ ${EXT_MSGPACK} = 'YES' ]; then \
    load_archive "https://github.com/msgpack/msgpack-php/archive/msgpack-${EXT_MSGPACK_VERSION}.tar.gz"; \
    cd /tmp/msgpack-php-msgpack-${EXT_MSGPACK_VERSION}; \
    make_ext; \
  fi; \
  if [ ${EXT_PHPREDIS} = 'YES' ]; then \
    load_archive "https://github.com/phpredis/phpredis/archive/${EXT_PHPREDIS_VERSION}.tar.gz"; \
    cd /tmp/phpredis-${EXT_PHPREDIS_VERSION}; \
    make_ext \
      --$(if_eq ${EXT_PHPREDIS_SESSION} 'YES' 'enable' 'disable')-redis-session \
      --$(if_eq ${EXT_PHPREDIS_JSON}    'YES' 'enable' 'disable')-redis-json \
      --$(if_eq ${EXT_PHPREDIS_MSGPACK} 'YES' 'enable' 'disable')-redis-msgpack \
        $(if_eq ${EXT_PHPREDIS_LZ4}     'YES' '--enable-redis-lz4  --with-liblz4=/usr') \
        $(if_eq ${EXT_PHPREDIS_ZSTD}    'YES' '--enable-redis-zstd --with-libzstd=/usr'); \
  fi; \
  if [ ${EXT_MEMCACHED} = 'YES' ]; then \
    load_archive "https://github.com/php-memcached-dev/php-memcached/archive/v${EXT_MEMCACHED_VERSION}.tar.gz"; \
    cd /tmp/php-memcached-${EXT_MEMCACHED_VERSION}; \
    make_ext \
      --disable-memcached-sasl \
      --$(if_eq ${EXT_MEMCACHED_SESSION} 'YES' 'enable' 'disable')-memcached-session \
      --$(if_eq ${EXT_MEMCACHED_JSON}    'YES' 'enable' 'disable')-memcached-json \
      --$(if_eq ${EXT_MEMCACHED_MSGPACK} 'YES' 'enable' 'disable')-memcached-msgpack \
      LDFLAGS="-lpthread"; \
  fi; \
  if [ ${EXT_MONGODB} = 'YES' ]; then \
    MONGODB_ARCHIVE="mongodb-${EXT_MONGODB_VERSION}"; \
    load_archive "https://github.com/mongodb/mongo-php-driver/releases/download/${EXT_MONGODB_VERSION}/${MONGODB_ARCHIVE}.tgz"; \
    cd /tmp/${MONGODB_ARCHIVE}; \
    make_ext \
      --with-mongodb-ssl=$(echo ${EXT_MONGODB_SSL} | tr '[:upper:]' '[:lower:]') \
      $(if_eq ${EXT_MONGODB_SSL} 'OPENSSL' --enable-mongodb-crypto-system-profile=yes); \
  fi; \ 
  if [ ${EXT_AMQP} = 'YES' ]; then \
    load_archive "https://github.com/php-amqp/php-amqp/archive/v${EXT_AMQP_VERSION}.tar.gz"; \
    cd /tmp/php-amqp-${EXT_AMQP_VERSION}; \
    make_ext \
      --with-librabbitmq-dir=/usr; \
  fi; \
  runtime_deps="$( \
    scanelf --needed --nobanner --recursive -F '%n#p' /usr/bin/php /usr/sbin/php-fpm /usr/lib/php/ \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .php-runtime ${runtime_deps}; \
  rm -rf \
    /usr/php/ \
    /usr/bin/phpize \
    /usr/bin/php-config \
    /usr/include/php/ \
    /usr/lib/php/build/; \
  : "-- Strip files"; \
  find /usr/lib/php/* -type f -name '*.a' -delete; \
  find \
    /usr/bin \
    /usr/sbin \
    /usr/lib/php/* \
    -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .php-build; \
  rm -rf /tmp/* /var/tmp/*

STOPSIGNAL SIGQUIT

CMD ["php"]
