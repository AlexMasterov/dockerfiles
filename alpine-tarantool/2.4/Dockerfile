FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-tarantool:2.4" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_TAG=2.4.2-1-g3f00d29
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_AVX=YES
ARG TARANTOOL_BACKTRACE=NO
ARG TARANTOOL_BUNDLED_LIBCURL=NO
ARG TARANTOOL_BUNDLED_LIBCURL_ARES=NO
ARG TARANTOOL_BUNDLED_LIBYAML=NO
ARG TARANTOOL_BUNDLED_ZSTD=NO
ARG TARANTOOL_DIST=NO

ARG LUAJIT_GC64=YES

ARG SSL_LIB=OPENSSL

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S tarantool; \
  adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN set -eux; \
  if_eq() { [ ${1} = ${2} ] && shift 2 && echo ${*:-}; }; \
  apk add --no-cache --virtual .tarantool-build \
    cmake \
    g++ \
    gcc \
    git \
    icu-dev \
    make \
    ncurses-dev \
    readline-dev \
    vim \
    `if_eq ${SSL_LIB}                   'OPENSSL'  openssl-dev` \
    `if_eq ${SSL_LIB}                   'LIBRESSL' libressl-dev` \
    `if_eq ${TARANTOOL_BACKTRACE}       'YES'      libunwind-dev` \
    `if_eq ${TARANTOOL_BUNDLED_LIBYAML} 'NO'       yaml-dev` \
    `if_eq ${TARANTOOL_BUNDLED_ZSTD}    'NO'       zstd-dev` \
    `if_eq ${TARANTOOL_BUNDLED_LIBCURL} 'YES' \
      autoconf \
    || echo \
      curl-dev \
    `; \
  : "-- Download Tarantool source"; \
  git clone --no-progress https://github.com/tarantool/tarantool.git /tmp/tarantool; \
  cd /tmp/tarantool; \
  git checkout ${TARANTOOL_TAG}; \
  : "-- Patch 'CMakeLists.txt': remove update_submodules()"; \
  sed -i 's/update_submodules()//g' CMakeLists.txt; \
  : "-- Git: update submodules"; \
  git \
    -c submodule."test-run".update=none \
    `if_eq ${TARANTOOL_BUNDLED_ZSTD}         'NO' -c submodule."third_party/zstd".update=none` \
    `if_eq ${TARANTOOL_BUNDLED_LIBYAML}      'NO' -c submodule."third_party/libyaml".update=none` \
    `if_eq ${TARANTOOL_BUNDLED_LIBCURL}      'NO' -c submodule."third_party/curl".update=none` \
    `if_eq ${TARANTOOL_BUNDLED_LIBCURL_ARES} 'NO' -c submodule."third_party/c-ares".update=none` \
    submodule update \
      --init \
      --recursive \
      --jobs $(getconf _NPROCESSORS_ONLN); \
  cmake \
    -DCMAKE_BUILD_TYPE="${TARANTOOL_BUILD_TYPE}" \
    -DENABLE_BACKTRACE:BOOL=${TARANTOOL_BACKTRACE} \
    -DENABLE_DIST:BOOL=${TARANTOOL_DIST} \
    -DENABLE_AVX:BOOL=${TARANTOOL_AVX} \
    -DENABLE_BUNDLED_LIBCURL:BOOL=${TARANTOOL_BUNDLED_LIBCURL} \
    -DENABLE_BUNDLED_LIBYAML:BOOL=${TARANTOOL_BUNDLED_LIBYAML} \
    -DENABLE_BUNDLED_ZSTD:BOOL=${TARANTOOL_BUNDLED_ZSTD} \
    -DLUAJIT_ENABLE_GC64:BOOL=${LUAJIT_GC64} \
    .; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/local/bin/tarantool \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .tarantool-runtime ${runtime_deps}; \
  : "-- Strip files"; \
  find /usr/lib/* -type f -name '*.a' -delete; \
  find \
    /usr/bin \
    /usr/sbin \
    /usr/local/bin \
    -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .tarantool-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["tarantool"]
