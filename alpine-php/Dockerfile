FROM alpine:3.14

LABEL repository.hub="alexmasterov/alpine-php" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 7.0.0 | 7.0
ARG PHP_VERSION=8.0

ARG PHP_CONFIG=/etc/php
ARG PHP_CFLAGS="-O2 -fPIC -fpie -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ARG PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ARG PHP_CLI=YES
ARG PHP_FPM=NO
ARG PHP_IPV6=NO

ARG EXT_BCMATH=NO
ARG EXT_BZ2=NO
ARG EXT_CTYPE=YES
ARG EXT_CURL=YES
ARG EXT_DOM=NO
ARG EXT_FILEINFO=NO
ARG EXT_FILTER=YES
ARG EXT_FTP=NO
ARG EXT_GD=NO
ARG EXT_GD_BUNDLED=YES
ARG EXT_GD_FREETYPE=YES
ARG EXT_GD_JPEG=YES
ARG EXT_GD_WEBP=YES
ARG EXT_GD_XPM=YES
ARG EXT_GMP=NO
ARG EXT_INTL=NO
ARG EXT_MBSTRING=YES
ARG EXT_MBSTRING_REGEX=YES
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
ARG EXT_SQLITE3=NO
ARG EXT_TOKENIZER=YES
ARG EXT_XML=NO
ARG EXT_XMLREADER=NO
ARG EXT_XMLWRITER=NO
ARG EXT_XSL=NO
ARG EXT_ZIP=NO
ARG EXT_ZIP_BUNDLED=NO
ARG EXT_ZLIB=NO

# < 7.4.0
ARG EXT_MBSTRING_REGEX_BUNDLED=NO
# >= 7.2.0
ARG PHP_ARGON2=YES
ARG EXT_SODIUM=YES
# >= 8.0.0
ARG EXT_FFI=NO
ARG EXT_OPCACHE_JIT=YES

ARG EXT_XDEBUG=NO
ARG EXT_XDEBUG_VERSION=3.0.4

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
ARG EXT_MONGODB_VERSION=1.10.0alpha1
ARG EXT_MONGODB_TLS=YES

ARG EXT_AMQP=NO
ARG EXT_AMQP_VERSION=1.11.0beta

ARG USE_TZ=NO
ARG LIB_SSL=OPENSSL

ARG OUT_STRIP=YES

RUN set -eux; \
  lib_ssl=$(echo ${LIB_SSL:-} | tr '[:upper:]' '[:lower:]'); \
  \
  if [ "${PHP_VERSION//[^.]}" = '.' ]; then \
    PHP_VERSION=$(wget "https://php.net/releases/index.php?json&version=${PHP_VERSION}" -T 30 -O - | sed -rn "s/.*\"version\":\s*?\"(.*\d+)\".*/\1/p"); \
    PHP_VERSION=${PHP_VERSION:?"Unknown PHP version: ${PHP_VERSION}"}; \
  fi; \
  \
  as_ver() { set -- ${1//./ }; local y="00${2:-0}" z="00${3:-0}"; echo "${1}${y:(-3)}${z:(-3)}"; }; \
  compare_ver() { test $(as_ver ${1}) -${2} $(as_ver ${3}); }; \
  contains() { if [ -z ${1##*${2}*} ]; then shift 2; echo ${*:-}; fi; }; \
  get_deps() { find ${*:-} -type f -and \( -executable -or -name '*.so*' \) -exec ldd '{}' ';' | sed -rn "s/\s*(.* => )?(.*) \(.*/\2/p" | sort -u; }; \
  if_eq() { if [ ${1} = ${2} ]; then shift 2; echo ${*:-}; fi; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo ${*:-}; fi; }; \
  if_php_ver() { if compare_ver ${PHP_VERSION} ${1} ${2}; then shift 2; echo ${*:-}; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo ${*:-}; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  make_ext() { phpize; ./configure --build=$(g++ -dumpmachine) ${*:-}; make; make install; }; \
  \
  apk add --no-cache --virtual .php-build \
    autoconf \
    bison \
    file \
    g++ \
    gcc \
    gnu-libiconv-dev \
    make \
    re2c \
    $(if_php_ver ge 7.1.17 \
      $(if_yes ${EXT_INTL} icu-dev) \
    ) \
    $(if_php_ver ge 7.2.0 \
      $(if_yes ${PHP_ARGON2} argon2-dev) \
      $(if_yes ${EXT_SODIUM} libsodium-dev) \
    ) \
    $(if_php_ver ge 8.0.0 \
      $(if_yes ${EXT_FFI} libffi-dev) \
    ) \
    $(if_no  ${EXT_PCRE_BUNDLED} pcre$(if_php_ver ge 7.3.0 2)-dev) \
    $(if_yes ${EXT_FFI}          libffi-dev) \
    $(if_yes ${EXT_BZ2}          bzip2-dev) \
    $(if_yes ${EXT_CURL}         curl-dev) \
    $(if_yes ${EXT_READLINE}     readline-dev) \
    $(if_yes ${EXT_GMP}          gmp-dev) \
    $(if_yes ${EXT_SQLITE3}      sqlite-dev) \
    $(if_yes ${EXT_POSTGRESQL}   postgresql-dev) \
    $(if_yes ${EXT_OPENSSL}      ${lib_ssl}-dev) \
    $(if_yes ${EXT_GD} \
      $(if_php_ver lt 7.4.0 \
        $(if_no ${EXT_GD_BUNDLED} \
          gd-dev \
          jpeg-dev \
          libxpm-dev \
        ) \
        $(if_yes ${EXT_GD_BUNDLED} \
          $(if_yes ${EXT_GD_JPEG} jpeg-dev) \
          $(if_yes ${EXT_GD_XPM}  libxpm-dev) \
        ) \
        $(if_yes ${EXT_GD_WEBP}     libwebp-dev) \
        $(if_yes ${EXT_GD_FREETYPE} freetype-dev) \
        libpng-dev \
      ) \
      $(if_php_ver ge 7.4.0 \
        $(if_no  ${EXT_GD_BUNDLED} gd-dev) \
        $(if_yes ${EXT_GD_BUNDLED} \
          $(if_no ${EXT_GD_JPEG}     jpeg-dev) \
          $(if_no ${EXT_GD_XPM}      libxpm-dev) \
          $(if_no ${EXT_GD_WEBP}     libwebp-dev) \
          $(if_no ${EXT_GD_FREETYPE} freetype-dev) \
          libpng-dev \
        ) \
      ) \
    ) \
    $(contains "${EXT_PDO_SQLITE}${EXT_PDO_PGSQL}${EXT_PDO_MYSQL}" 'YES' \
      $(if_yes ${EXT_PDO_SQLITE} sqlite-dev) \
      $(if_yes ${EXT_PDO_PGSQL}  postgresql-dev) \
      $(if_yes ${EXT_PDO_MYSQL} \
        $(if_no ${EXT_PDO_MYSQL_BUNDLED} mariadb-dev) \
      ) \
    ) \
    $(contains "${EXT_DOM}${EXT_SIMPLEXML}${EXT_XML}${EXT_XMLREADER}${EXT_XMLWRITER}${EXT_XSL}" 'YES' \
      $(if_yes ${EXT_XSL} libxslt-dev) \
      libxml2-dev \
    ) \
    $(if_yes ${EXT_ZLIB} zlib-dev) \
    $(if_yes ${EXT_ZIP} \
      $(if_no ${EXT_ZIP_BUNDLED} libzip-dev) \
      zlib-dev \
    ) \
    $(if_yes ${EXT_MBSTRING} \
      $(if_yes ${EXT_MBSTRING_REGEX} \
        $(if_php_ver lt 7.4.0 $(if_no ${EXT_MBSTRING_REGEX_BUNDLED} oniguruma-dev)) \
        $(if_php_ver ge 7.4.0 oniguruma-dev) \
      ) \
    ) \
    \
    $(if_yes ${EXT_AMQP} rabbitmq-c-dev) \
    $(if_yes ${EXT_MEMCACHED} \
      libmemcached-dev \
      zlib-dev \
    ) \
    $(if_yes ${EXT_PHPREDIS} \
      $(if_yes ${EXT_PHPREDIS_LZ4}  lz4-dev) \
      $(if_yes ${EXT_PHPREDIS_ZSTD} zstd-dev) \
    ) \
    $(if_yes ${EXT_MONGODB} \
      $(if_yes ${EXT_MONGODB_TLS} ${lib_ssl}-dev) \
    ) \
    ; \
  if compare_ver ${PHP_VERSION} le 7.1.16; then \
    if [ ${EXT_INTL} = 'YES' ]; then \
      apk add --no-cache --virtual .php-build-intl --repository http://dl-cdn.alpinelinux.org/alpine/v3.8/main/ \
        icu-dev=60.2-r3; \
    fi; \
    if [ "${EXT_MBSTRING}${EXT_MBSTRING_REGEX}${EXT_MBSTRING_REGEX_BUNDLED}" = 'YESYESNO' ]; then \
      apk add --no-cache --virtual .php-build-mbregex --repository http://dl-cdn.alpinelinux.org/alpine/v3.7/main/ \
        oniguruma-dev=6.6.1-r0; \
    fi; \
  fi; \
  if [ ${USE_TZ} = 'YES' ]; then \
    apk add --no-cache --virtual .php-tz tzdata; \
  fi; \
  : "-- GNU iconv: replace binary and headers"; \
  mv /usr/bin/gnu-iconv /usr/bin/iconv; \
  mv /usr/include/gnu-libiconv/*.h /usr/include; \
  : "-- make: setup flags"; \
  export MAKEFLAGS="-j $(( $(getconf _NPROCESSORS_ONLN) - 1 ))"; \
  : "-- PHP: download source"; \
  load_archive "https://www.php.net/distributions/php-${PHP_VERSION}.tar.xz" xJ; \
  cd /tmp/php-${PHP_VERSION}; \
  : "-- PHP: build"; \
  ./configure \
    --build=$(g++ -dumpmachine) \
    --datadir=/tmp \
    --datarootdir=/tmp \
    --prefix=/usr/local \
    --sysconfdir=${PHP_CONFIG} \
    --with-config-file-path=${PHP_CONFIG} \
    --with-config-file-scan-dir=${PHP_CONFIG}/conf.d \
    --with-layout=GNU \
    --without-pear \
    --disable-cgi \
    --disable-debug \
    --disable-phpdbg \
    --disable-rpath \
    --enable-option-checking=fatal \
    --enable-fd-setsize=$(ulimit -n) \
    --with-mhash \
    --with-iconv=/usr \
    $(if_no  ${PHP_CLI}           --disable-cli) \
    $(if_no  ${PHP_IPV6}          --disable-ipv6) \
    $(if_yes ${PHP_FPM}           --enable-fpm) \
    $(if_php_ver ge 7.2.0 \
      $(if_yes ${PHP_ARGON2}      --with-password-argon2) \
      $(if_yes ${EXT_SODIUM}      --with-sodium=/usr) \
    ) \
    $(if_php_ver ge 8.0.0 \
      $(if_yes ${EXT_FFI}         --with-ffi) \
      $(if_no  ${EXT_OPCACHE_JIT} --disable-opcache-jit) \
    ) \
    $(if_no  ${EXT_POSIX}         --disable-posix) \
    $(if_no  ${EXT_CTYPE}         --disable-ctype) \
    $(if_no  ${EXT_PHAR}          --disable-phar) \
    $(if_no  ${EXT_FILEINFO}      --disable-fileinfo) \
    $(if_no  ${EXT_FILTER}        --disable-filter) \
    $(if_no  ${EXT_TOKENIZER}     --disable-tokenizer) \
    $(if_yes ${EXT_INTL}          --enable-intl) \
    $(if_yes ${EXT_PCNTL}         --enable-pcntl) \
    $(if_yes ${EXT_FTP}           --enable-ftp) \
    $(if_yes ${EXT_BCMATH}        --enable-bcmath) \
    $(if_yes ${EXT_CURL}          --with-curl=/usr) \
    $(if_yes ${EXT_GMP}           --with-gmp=/usr) \
    $(if_yes ${EXT_BZ2}           --with-bz2=/usr) \
    $(if_yes ${EXT_READLINE}      --with-readline=/usr) \
    $(if_no  ${EXT_SQLITE3}       --without-sqlite3) \
    $(if_yes ${EXT_SQLITE3}       --with-sqlite3=/usr) \
    $(if_no  ${EXT_PCRE_BUNDLED} \
      $(if_php_ver lt 7.4.0 --with-pcre-regex=/usr) \
      $(if_php_ver ge 7.4.0 --with-external-pcre=/usr) \
    ) \
    $(if_yes ${EXT_OPENSSL} \
      $(if_eq "${LIB_SSL}${EXT_OPENSSL_SYSTEM_CIPHERS}" 'opensslYES' --with-system-ciphers) \
      --with-openssl=/usr \
    ) \
    $(if_yes ${EXT_ZLIB} --with-zlib=/usr) \
    $(if_yes ${EXT_ZIP} \
      $(if_no ${EXT_ZIP_BUNDLED} --with-libzip=/usr) \
      --enable-zip \
      --with-zlib=/usr \
    ) \
    $(contains "${EXT_DOM}${EXT_SIMPLEXML}${EXT_XML}${EXT_XMLREADER}${EXT_XMLWRITER}${EXT_XSL}" 'YES' \
      $(if_no  ${EXT_DOM}       --disable-dom) \
      $(if_no  ${EXT_XML}       --disable-xml) \
      $(if_no  ${EXT_XMLREADER} --disable-xmlreader) \
      $(if_no  ${EXT_XMLWRITER} --disable-xmlwriter) \
      $(if_no  ${EXT_SIMPLEXML} --disable-simplexml) \
      $(if_yes ${EXT_XSL}       --with-xsl=/usr) \
    ) \
    $(if_eq "${EXT_DOM}${EXT_SIMPLEXML}${EXT_XML}${EXT_XMLREADER}${EXT_XMLWRITER}${EXT_XSL}" 'NONONONONONO' \
      $(if_php_ver lt 7.4.0 --disable-libxml) \
      $(if_php_ver ge 7.4.0 --without-libxml) \
      --disable-dom \
      --disable-xml \
      --disable-xmlreader \
      --disable-xmlwriter \
      --disable-simplexml \
    ) \
    $(if_eq "${EXT_PDO_SQLITE}${EXT_PDO_PGSQL}${EXT_PDO_MYSQL}" 'NONONO' \
      --disable-pdo \
    ) \
    $(contains "${EXT_PDO_SQLITE}${EXT_PDO_PGSQL}${EXT_PDO_MYSQL}" 'YES' \
      $(if_no  ${EXT_PDO_SQLITE} --without-pdo-sqlite) \
      $(if_yes ${EXT_PDO_SQLITE} --with-pdo-sqlite=/usr) \
      $(if_yes ${EXT_PDO_PGSQL}  --with-pdo-pgsql=/usr) \
      $(if_yes ${EXT_PDO_MYSQL} \
        $(if_no  ${EXT_PDO_MYSQL_BUNDLED} --with-pdo-mysql=/usr) \
        $(if_yes ${EXT_PDO_MYSQL_BUNDLED} --with-pdo-mysql=mysqlnd) \
      ) \
    ) \
    $(if_yes ${EXT_GD} \
      $(if_php_ver lt 7.4.0 \
        $(if_no ${EXT_GD_BUNDLED} \
          --with-gd=/usr \
          --with-jpeg-dir=/usr \
          --with-xpm-dir=/usr \
        ) \
        $(if_yes ${EXT_GD_BUNDLED} \
          $(if_yes ${EXT_GD_JPEG} --with-jpeg-dir=/usr) \
          $(if_yes ${EXT_GD_XPM}  --with-xpm-dir=/usr) \
          --with-gd \
        ) \
        $(if_yes ${EXT_GD_WEBP}     --with-webp-dir=/usr) \
        $(if_yes ${EXT_GD_FREETYPE} --with-freetype-dir=/usr) \
        --with-png-dir=/usr \
      ) \
      $(if_php_ver ge 7.4.0 \
        $(if_no ${EXT_GD_BUNDLED} --with-external-gd=/usr) \
        --enable-gd \
      ) \
    ) \
    $(if_yes ${EXT_MBSTRING} \
      $(if_no  ${EXT_MBSTRING_REGEX} --disable-mbregex) \
      $(if_yes ${EXT_MBSTRING_REGEX} \
        $(if_php_ver lt 7.4.0 $(if_no ${EXT_MBSTRING_REGEX_BUNDLED} --with-onig=/usr)) \
        --enable-mbregex \
      ) \
      --enable-mbstring \
    ) \
    CFLAGS="${PHP_CFLAGS:-}" \
    CPPFLAGS="${PHP_CFLAGS:-}" \
    LDFLAGS="${PHP_LDFLAGS:-}" \
    ; \
  make; \
  make install; \
  \
  ext_config_dir=$(php --ini | sed -rn "s/.*\.ini files in:\s*(.*)/\1/p"); \
  [ -d ${ext_config_dir} ] || mkdir -p ${ext_config_dir}; \
  \
  echo "zend_extension=opcache.so" >> ${ext_config_dir}/01-opcache.ini; \
  \
  if [ ${EXT_XDEBUG} = 'YES' ]; then \
    case 1 in \
      $(if_php_ver lt 7.1.0 1)) EXT_XDEBUG_VERSION=2.7.2 ;; \
      $(if_php_ver lt 7.2.0 1)) EXT_XDEBUG_VERSION=2.9.8 ;; \
    esac; \
    load_archive "https://github.com/xdebug/xdebug/archive/${EXT_XDEBUG_VERSION}.tar.gz"; \
    cd /tmp/xdebug-${EXT_XDEBUG_VERSION}; \
    make_ext; \
    echo "zend_extension=xdebug.so" >> ${ext_config_dir}/01-xdebug.ini; \
  fi; \
  if [ ${EXT_MSGPACK} = 'YES' ]; then \
    load_archive "https://github.com/msgpack/msgpack-php/archive/msgpack-${EXT_MSGPACK_VERSION}.tar.gz"; \
    cd /tmp/msgpack-php-msgpack-${EXT_MSGPACK_VERSION}; \
    make_ext; \
    echo "extension=msgpack.so" >> ${ext_config_dir}/02-msgpack.ini; \
  fi; \
  if [ ${EXT_APCU} = 'YES' ]; then \
    load_archive "https://github.com/krakjoe/apcu/archive/v${EXT_APCU_VERSION}.tar.gz"; \
    cd /tmp/apcu-${EXT_APCU_VERSION}; \
    make_ext \
      --$(if_yes ${EXT_APCU_RWLOCKS}   enable)$(if_no ${EXT_APCU_RWLOCKS}   disable)-apcu-rwlocks \
      --$(if_yes ${EXT_APCU_SPINLOCKS} enable)$(if_no ${EXT_APCU_SPINLOCKS} disable)-apcu-spinlocks; \
    echo "extension=apcu.so" >> ${ext_config_dir}/03-apcu.ini; \
  fi; \
  if [ ${EXT_PHPREDIS} = 'YES' ]; then \
    load_archive "https://github.com/phpredis/phpredis/archive/${EXT_PHPREDIS_VERSION}.tar.gz"; \
    cd /tmp/phpredis-${EXT_PHPREDIS_VERSION}; \
    make_ext \
      $(if_yes ${EXT_PHPREDIS_LZ4}  --enable-redis-lz4  --with-liblz4=/usr) \
      $(if_yes ${EXT_PHPREDIS_ZSTD} --enable-redis-zstd --with-libzstd=/usr) \
      --$(if_yes ${EXT_PHPREDIS_SESSION} enable)$(if_no ${EXT_PHPREDIS_SESSION} disable)-redis-session \
      --$(if_yes ${EXT_PHPREDIS_JSON}    enable)$(if_no ${EXT_PHPREDIS_JSON}    disable)-redis-json \
      $(if_yes ${EXT_PHPREDIS_MSGPACK} \
        $(if_yes ${EXT_MSGPACK} --enable-redis-msgpack) \
      ); \
    echo "extension=redis.so" >> ${ext_config_dir}/03-redis.ini; \
  fi; \
  if [ ${EXT_MEMCACHED} = 'YES' ]; then \
    load_archive "https://github.com/php-memcached-dev/php-memcached/archive/v${EXT_MEMCACHED_VERSION}.tar.gz"; \
    cd /tmp/php-memcached-${EXT_MEMCACHED_VERSION}; \
    make_ext \
      --disable-memcached-sasl \
      --with-zlib-dir=/usr \
      --$(if_yes ${EXT_MEMCACHED_SESSION} enable)$(if_no ${EXT_MEMCACHED_SESSION} disable)-memcached-session \
      --$(if_yes ${EXT_MEMCACHED_JSON}    enable)$(if_no ${EXT_MEMCACHED_JSON}    disable)-memcached-json \
      $(if_yes ${EXT_MEMCACHED_MSGPACK} \
        $(if_yes ${EXT_MSGPACK} --enable-memcached-msgpack) \
      ); \
    echo "extension=memcached.so" >> ${ext_config_dir}/03-memcached.ini; \
  fi; \
  if [ ${EXT_MONGODB} = 'YES' ]; then \
    MONGODB_ARCHIVE="mongodb-${EXT_MONGODB_VERSION}"; \
    load_archive "https://github.com/mongodb/mongo-php-driver/releases/download/${EXT_MONGODB_VERSION}/${MONGODB_ARCHIVE}.tgz"; \
    cd /tmp/${MONGODB_ARCHIVE}; \
    make_ext \
      --with-mongodb-sasl=no \
      --with-mongodb-ssl=$(if_yes ${EXT_MONGODB_TLS} ${lib_ssl})$(if_no ${EXT_MONGODB_TLS} no) \
      $(if_eq "${EXT_MONGODB_TLS}${lib_ssl}${EXT_OPENSSL_SYSTEM_CIPHERS}" 'YESopensslYES' \
        --enable-mongodb-crypto-system-profile=on; \
      ); \
    echo "extension=mongodb.so" >> ${ext_config_dir}/03-mongodb.ini; \
  fi; \
  if [ ${EXT_AMQP} = 'YES' ]; then \
    load_archive "https://github.com/php-amqp/php-amqp/archive/v${EXT_AMQP_VERSION}.tar.gz"; \
    cd /tmp/php-amqp-${EXT_AMQP_VERSION}; \
    make_ext \
      --with-librabbitmq-dir=/usr; \
    echo "extension=amqp.so" >> ${ext_config_dir}/03-amqp.ini; \
  fi; \
  \
  : "-- Out: clean up"; \
  find /usr/local/lib/php -depth \( -type f -not -name '*.so' -or -type d -empty \) -delete; \
  \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    : "-- Out: strip executables"; \
    find /usr/local -type f -executable \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  runtime_deps=$(get_deps /usr/local/bin /usr/local/lib | sed -rn "s/\/usr\/lib\/(.*)/so:\1/p"); \
  apk add --no-cache --virtual .php-runtime ${runtime_deps}; \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .php-build*; \
  rm -rf /tmp/* /var/tmp/*

STOPSIGNAL SIGQUIT

CMD ["php"]
