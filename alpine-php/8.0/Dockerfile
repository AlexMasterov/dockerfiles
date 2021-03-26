FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-php:8.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG PHP_VERSION=8.0.3
ARG PHP_CONFIG=/etc/php

ARG PHP_CFLAGS="-march=native -O2 -fpic -fpie -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ARG PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ARG LIB_SSL=OPENSSL

ARG PHP_ARGON2=YES
ARG PHP_CLI=YES
ARG PHP_FPM=NO
ARG PHP_IPV6=NO

ARG EXT_BCMATH=NO
ARG EXT_BZ2=NO
ARG EXT_CTYPE=YES
ARG EXT_CURL=YES
ARG EXT_DOM=NO
ARG EXT_FFI=NO
ARG EXT_FILEINFO=NO
ARG EXT_FILTER=YES
ARG EXT_FTP=NO
ARG EXT_GD=NO
ARG EXT_GD_BUNDLED=YES
ARG EXT_GD_FREETYPE_BUNDLED=YES
ARG EXT_GD_JPEG_BUNDLED=YES
ARG EXT_GD_WEBP_BUNDLED=YES
ARG EXT_GD_XPM_BUNDLED=YES
ARG EXT_GMP=NO
ARG EXT_INTL=NO
ARG EXT_MBSTRING=YES
ARG EXT_MBSTRING_REGEX=YES
ARG EXT_OPCACHE_JIT=YES
ARG EXT_OPENSSL=YES
ARG EXT_OPENSSL_SYSTEM_CIPHERS=YES
ARG EXT_PCNTL=NO
ARG EXT_PCRE_BUNDLED=NO
ARG EXT_PDO_MYSQL=NO
ARG EXT_PDO_MYSQL_BUNDLED=NO
ARG EXT_PDO_PGSQL=NO
ARG EXT_PDO_SQLITE=YES
ARG EXT_PHAR=NO
ARG EXT_POSIX=NO
ARG EXT_POSTGRESQL=NO
ARG EXT_READLINE=NO
ARG EXT_SIMPLEXML=YES
ARG EXT_SODIUM=YES
ARG EXT_SQLITE3=NO
ARG EXT_TOKENIZER=YES
ARG EXT_XML=NO
ARG EXT_XMLREADER=NO
ARG EXT_XMLWRITER=NO
ARG EXT_XSL=NO
ARG EXT_ZIP=NO
ARG EXT_ZIP_BUNDLED=NO
ARG EXT_ZLIB=NO

ARG EXT_XDEBUG=NO
ARG EXT_XDEBUG_VERSION=3.0.3

ARG EXT_APCU=NO
ARG EXT_APCU_VERSION=5.1.20
ARG EXT_APCU_RWLOCKS=YES
ARG EXT_APCU_SPINLOCKS=NO

ARG EXT_MSGPACK=NO
ARG EXT_MSGPACK_VERSION=2.1.2

ARG EXT_PHPREDIS=NO
ARG EXT_PHPREDIS_VERSION=5.3.4
ARG EXT_PHPREDIS_SESSION=YES
ARG EXT_PHPREDIS_JSON=YES
ARG EXT_PHPREDIS_MSGPACK=NO
ARG EXT_PHPREDIS_LZ4=NO
ARG EXT_PHPREDIS_ZSTD=NO

ARG EXT_MEMCACHED=NO
ARG EXT_MEMCACHED_VERSION=3.1.5
ARG EXT_MEMCACHED_SESSION=YES
ARG EXT_MEMCACHED_JSON=YES
ARG EXT_MEMCACHED_MSGPACK=NO

ARG EXT_MONGODB=NO
ARG EXT_MONGODB_VERSION=1.9.0
ARG EXT_MONGODB_TLS=YES

ARG EXT_AMQP=NO
ARG EXT_AMQP_VERSION=1.11.0beta

RUN set -eux; \
  addgroup -g 82 -S www-data; \
  adduser -u 82 -S -D -h /var/cache/www-data -s /sbin/nologin -G www-data www-data

RUN set -eux; \
  if_eq() { [ ${1} = ${2} ] && shift 2 && echo "${*:-}"; }; \
  if_no() { [ ${1} = 'NO' ] && shift 1 && echo "${*:-}"; }; \
  if_yes() { [ ${1} = 'YES' ] && shift 1 && echo "${*:-}"; }; \
  contains() { [ -z ${1##*${2}*} ] && shift 2 && echo "${*:-}"; }; \
  to_lowercase() { echo ${1:-} | tr '[:upper:]' '[:lower:]'; }; \
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
    $(if_yes ${EXT_FFI}          libffi-dev) \
    $(if_yes ${EXT_SODIUM}       libsodium-dev) \
    $(if_yes ${PHP_ARGON2}       argon2-dev) \
    $(if_yes ${EXT_INTL}         icu-dev) \
    $(if_no  ${EXT_PCRE_BUNDLED} pcre2-dev) \
    $(if_yes ${EXT_BZ2}          bzip2-dev) \
    $(if_yes ${EXT_CURL}         curl-dev) \
    $(if_yes ${EXT_READLINE}     readline-dev) \
    $(if_yes ${EXT_GMP}          gmp-dev) \
    $(if_yes ${EXT_SQLITE3}      sqlite-dev) \
    $(if_yes ${EXT_POSTGRESQL}   postgresql-dev) \
    $(if_yes ${EXT_OPENSSL}      $(to_lowercase ${LIB_SSL})-dev) \
    $(if_eq "${EXT_GD}${EXT_GD_BUNDLED}" 'YESNO' gd-dev) \
    $(if_eq "${EXT_GD}${EXT_GD_BUNDLED}" 'YESYES' \
      libpng-dev \
      $(if_no ${EXT_GD_JPEG_BUNDLED}     jpeg-dev) \
      $(if_no ${EXT_GD_XPM_BUNDLED}      libxpm-dev) \
      $(if_no ${EXT_GD_WEBP_BUNDLED}     libwebp-dev) \
      $(if_no ${EXT_GD_FREETYPE_BUNDLED} freetype-dev) \
    ) \
    $(contains "${EXT_PDO_SQLITE}${EXT_PDO_PGSQL}${EXT_PDO_MYSQL}" 'YES' \
      $(if_eq "${EXT_PDO_MYSQL}${EXT_PDO_MYSQL_BUNDLED}" 'YESNO' mariadb-dev) \
      $(if_yes ${EXT_PDO_PGSQL}  postgresql-dev) \
      $(if_yes ${EXT_PDO_SQLITE} sqlite-dev) \
    ) \
    $(contains "${EXT_DOM}${EXT_SIMPLEXML}${EXT_XML}${EXT_XMLREADER}${EXT_XMLWRITER}${EXT_XSL}" 'YES' \
      libxml2-dev \
      $(if_yes ${EXT_XSL} libxslt-dev) \
    ) \
    $(contains "${EXT_ZLIB}${EXT_ZIP}" 'YES' zlib-dev) \
    $(if_eq "${EXT_ZIP}${EXT_ZIP_BUNDLED}" 'YESNO' libzip-dev) \
    $(if_eq "${EXT_MBSTRING}${EXT_MBSTRING_REGEX}" 'YESYES' oniguruma-dev) \
    \
    $(if_yes ${EXT_MEMCACHED}     libmemcached-dev) \
    $(if_yes ${EXT_AMQP}          rabbitmq-c-dev) \
    $(if_yes ${EXT_PHPREDIS} \
      $(if_yes ${EXT_PHPREDIS_LZ4}  lz4-dev) \
      $(if_yes ${EXT_PHPREDIS_ZSTD} zstd-dev) \
    ) \
    $(if_eq "${EXT_MONGODB}${EXT_MONGODB_TLS}" 'YESYES' $(to_lowercase ${LIB_SSL})-dev) \
    ; \
  : "-- GNU iconv: replace binary and headers"; \
  mv /usr/bin/gnu-iconv /usr/bin/iconv; \
  mv /usr/include/gnu-libiconv/*.h /usr/include; \
  : "-- Setup make flags"; \
  MAKEFLAGS="-j $(( $(getconf _NPROCESSORS_ONLN) - 1 ))"; \
  : "-- PHP: download"; \
  load_archive "https://www.php.net/distributions/php-${PHP_VERSION}.tar.xz" xJ; \
  cd /tmp/php-${PHP_VERSION}; \
  : "-- PHP: build"; \
  ./configure \
    --prefix=/usr \
    --datadir=/tmp \
    --datarootdir=/tmp \
    --sysconfdir="${PHP_CONFIG}" \
    --with-config-file-path="${PHP_CONFIG}" \
    --with-config-file-scan-dir="${PHP_CONFIG}/conf.d" \
    --with-layout="GNU" \
    --without-pear \
    --disable-cgi \
    --disable-debug \
    --disable-phpdbg \
    --disable-rpath \
    --enable-option-checking=fatal \
    --enable-fd-setsize=$(ulimit -n) \
    --with-mhash \
    --with-iconv=/usr \
    $(if_no  ${EXT_OPCACHE_JIT}  --disable-opcache-jit) \
    $(if_yes ${EXT_FFI}          --with-ffi) \
    $(if_yes ${EXT_SODIUM}       --with-sodium=/usr) \
    $(if_yes ${PHP_ARGON2}       --with-password-argon2) \
    $(if_yes ${PHP_FPM}          --enable-fpm) \
    $(if_no  ${PHP_CLI}          --disable-cli) \
    $(if_no  ${PHP_IPV6}         --disable-ipv6) \
    $(if_no  ${EXT_POSIX}        --disable-posix) \
    $(if_no  ${EXT_CTYPE}        --disable-ctype) \
    $(if_no  ${EXT_PHAR}         --disable-phar) \
    $(if_no  ${EXT_FILEINFO}     --disable-fileinfo) \
    $(if_no  ${EXT_FILTER}       --disable-filter) \
    $(if_no  ${EXT_TOKENIZER}    --disable-tokenizer) \
    $(if_no  ${EXT_PCRE_BUNDLED} --with-external-pcre=/usr) \
    $(if_yes ${EXT_INTL}         --enable-intl) \
    $(if_yes ${EXT_PCNTL}        --enable-pcntl) \
    $(if_yes ${EXT_FTP}          --enable-ftp) \
    $(if_yes ${EXT_BCMATH}       --enable-bcmath) \
    $(if_yes ${EXT_CURL}         --with-curl=/usr) \
    $(if_yes ${EXT_GMP}          --with-gmp=/usr) \
    $(if_yes ${EXT_BZ2}          --with-bz2=/usr) \
    $(if_yes ${EXT_READLINE}     --with-readline=/usr) \
    $(if_yes ${EXT_SQLITE3}      --with-sqlite3=/usr || echo --without-sqlite3) \
    $(if_yes ${EXT_OPENSSL} \
      --with-openssl=/usr \
      $(if_eq "${LIB_SSL}${EXT_OPENSSL_SYSTEM_CIPHERS}" 'OPENSSLYES' --with-system-ciphers) \
    ) \
    $(if_yes ${EXT_ZIP} \
      --enable-zip \
      $(if_no ${EXT_ZIP_BUNDLED} --with-libzip=/usr) \
    ) \
    $(contains "${EXT_ZLIB}${EXT_ZIP}" 'YES' --with-zlib=/usr) \
    $(contains "${EXT_DOM}${EXT_SIMPLEXML}${EXT_XML}${EXT_XMLREADER}${EXT_XMLWRITER}${EXT_XSL}" 'YES' \
      $(if_no  ${EXT_DOM}       --disable-dom) \
      $(if_no  ${EXT_XML}       --disable-xml) \
      $(if_no  ${EXT_XMLREADER} --disable-xmlreader) \
      $(if_no  ${EXT_XMLWRITER} --disable-xmlwriter) \
      $(if_no  ${EXT_SIMPLEXML} --disable-simplexml) \
      $(if_yes ${EXT_XSL}       --with-xsl=/usr) \
    || echo \
      --without-libxml \
      --disable-dom \
      --disable-xml \
      --disable-xmlreader \
      --disable-xmlwriter \
      --disable-simplexml \
    ) \
    $(contains "${EXT_PDO_SQLITE}${EXT_PDO_PGSQL}${EXT_PDO_MYSQL}" 'YES' \
      $(if_yes ${EXT_PDO_SQLITE} --with-pdo-sqlite=/usr || echo --without-pdo-sqlite) \
      $(if_yes ${EXT_PDO_PGSQL}  --with-pdo-pgsql=/usr) \
      $(if_yes ${EXT_PDO_MYSQL} \
        $(if_yes ${EXT_PDO_MYSQL_BUNDLED} --with-pdo-mysql=mysqlnd || echo --with-pdo-mysql=/usr) \
      ) \
    || echo \
      --disable-pdo \
    ) \
    $(if_yes ${EXT_GD} \
      --enable-gd \
      $(if_no ${EXT_GD_BUNDLED} --with-external-gd=/usr) \
    ) \
    $(if_yes ${EXT_MBSTRING} \
      --enable-mbstring \
      --$(if_yes ${EXT_MBSTRING_REGEX} 'enable' || echo 'disable')-mbregex \
    ) \
    CFLAGS="${PHP_CFLAGS}" \
    CPPFLAGS="${PHP_CFLAGS}" \
    LDFLAGS="${PHP_LDFLAGS}"; \
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
      --$(if_yes ${EXT_APCU_RWLOCKS}   'enable' || echo 'disable')-apcu-rwlocks \
      --$(if_yes ${EXT_APCU_SPINLOCKS} 'enable' || echo 'disable')-apcu-spinlocks; \
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
      $(if_yes ${EXT_PHPREDIS_LZ4}  --enable-redis-lz4  --with-liblz4=/usr) \
      $(if_yes ${EXT_PHPREDIS_ZSTD} --enable-redis-zstd --with-libzstd=/usr) \
      --$(if_yes ${EXT_PHPREDIS_SESSION} 'enable' || echo 'disable')-redis-session \
      --$(if_yes ${EXT_PHPREDIS_JSON}    'enable' || echo 'disable')-redis-json \
      --$(if_eq "${EXT_MSGPACK}${EXT_PHPREDIS_MSGPACK}" 'YESYES' 'enable' || echo 'disable')-redis-msgpack; \
  fi; \
  if [ ${EXT_MEMCACHED} = 'YES' ]; then \
    load_archive "https://github.com/php-memcached-dev/php-memcached/archive/v${EXT_MEMCACHED_VERSION}.tar.gz"; \
    cd /tmp/php-memcached-${EXT_MEMCACHED_VERSION}; \
    make_ext \
      EXTRA_LDFLAGS="-lpthread" \
      --disable-memcached-sasl \
      --$(if_yes ${EXT_MEMCACHED_SESSION} 'enable' || echo 'disable')-memcached-session \
      --$(if_yes ${EXT_MEMCACHED_JSON}    'enable' || echo 'disable')-memcached-json \
      --$(if_eq "${EXT_MSGPACK}${EXT_MEMCACHED_MSGPACK}" 'YESYES' 'enable' || echo 'disable')-memcached-msgpack; \
  fi; \
  if [ ${EXT_MONGODB} = 'YES' ]; then \
    MONGODB_ARCHIVE="mongodb-${EXT_MONGODB_VERSION}"; \
    load_archive "https://github.com/mongodb/mongo-php-driver/releases/download/${EXT_MONGODB_VERSION}/${MONGODB_ARCHIVE}.tgz"; \
    cd /tmp/${MONGODB_ARCHIVE}; \
    make_ext \
      --with-mongodb-sasl=no \
      --with-mongodb-ssl=$(if_yes ${EXT_MONGODB_TLS} $(to_lowercase ${LIB_SSL}) || echo 'no') \
      $(if_eq "${EXT_MONGODB_TLS}${LIB_SSL}${EXT_OPENSSL_SYSTEM_CIPHERS}" 'YESOPENSSLYES' \
        --enable-mongodb-crypto-system-profile=on; \
      ); \
  fi; \
  if [ ${EXT_AMQP} = 'YES' ]; then \
    load_archive "https://github.com/php-amqp/php-amqp/archive/v${EXT_AMQP_VERSION}.tar.gz"; \
    cd /tmp/php-amqp-${EXT_AMQP_VERSION}; \
    make_ext \
      --with-librabbitmq-dir=/usr; \
  fi; \
  : "-- Strip files"; \
  rm -rf \
    /usr/share/man \
    /usr/include/* \
    /usr/bin/phpize \
    /usr/bin/php-config \
    /usr/lib/php/build/ \
    /usr/lib/php/**/*.a; \
  find \
    /usr/bin \
    /usr/sbin \
    /usr/lib \
    -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  runtime_deps="$( \
    scanelf --needed --nobanner --recursive -F '%n#p' /usr/bin/php /usr/sbin/php-fpm /usr/lib/php/ \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .php-runtime ${runtime_deps}; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .php-build; \
  rm -rf /tmp/* /var/tmp/*

STOPSIGNAL SIGQUIT

CMD ["php"]
