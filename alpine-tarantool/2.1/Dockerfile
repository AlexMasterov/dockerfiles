FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-tarantool:2.1" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_TAG=2.1.3-0-g38d7d6c
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=ON
ARG TARANTOOL_DIST=ON
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_BUNDLED_LUAJIT=ON
ARG TARANTOOL_BUNDLED_LIBCURL=OFF
ARG TARANTOOL_BUNDLED_LIBYAML=OFF
ARG TARANTOOL_WITH_SYSTEMD=OFF
ARG TARANTOOL_WITH_SYSVINIT=ON
ARG TARANTOOL_WITH_LOGROTATE=ON

ARG LUAJIT_GC64=ON

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S tarantool; \
  adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN set -eux; \
  apk add --no-cache --virtual .tarantool-build \
    autoconf \
    automake \
    binutils-dev \
    cmake \
    curl-dev \
    g++ \
    gcc \
    git \
    icu-dev \
    libressl-dev \
    libtool \
    make \
    ncurses-dev \
    readline-dev \
    $(test "${TARANTOOL_BACKTRACE}" = 'ON' && echo 'libunwind-dev') \
    $(test "${TARANTOOL_BUNDLED_LIBCURL}" = 'OFF' && echo 'curl-dev') \
    $(test "${TARANTOOL_BUNDLED_LIBYAML}" = 'OFF' && echo 'yaml-dev'); \
  : "-- Download Tarantool source"; \
  git clone https://github.com/tarantool/tarantool.git /tmp/tarantool; \
  cd /tmp/tarantool; \
  git checkout ${TARANTOOL_TAG}; \
  : "-- Patch 'CMakeLists.txt': remove update_submodules"; \
  sed -i 's/update_submodules()//g' CMakeLists.txt; \
  : "-- Git: update submodules"; \
  git \
    $(test "${TARANTOOL_BUNDLED_LIBCURL}" = 'OFF' && \
        echo \ 
          -c submodule."third_party/curl".update=none \
          -c submodule."third_party/c-ares".update=none) \
    $(test "${TARANTOOL_BUNDLED_LIBYAML}" = 'OFF' && \
        echo -c submodule."third_party/libyaml".update=none) \
    submodule update \
      --init \
      --recursive \
      --single-branch \
      --jobs $(getconf _NPROCESSORS_ONLN); \
  : "-- Build Tarantool"; \
  cmake \
    -DCMAKE_BUILD_TYPE=${TARANTOOL_BUILD_TYPE} \
    -DENABLE_BACKTRACE:BOOL=${TARANTOOL_BACKTRACE} \
    -DENABLE_AVX:BOOL=${TARANTOOL_AVX} \
    -DENABLE_DIST:BOOL=${TARANTOOL_DIST} \
    -DENABLE_BUNDLED_LUAJIT:BOOL=${TARANTOOL_BUNDLED_LUAJIT} \
    -DENABLE_BUNDLED_LIBCURL:BOOL=${TARANTOOL_BUNDLED_LIBCURL} \
    -DENABLE_BUNDLED_LIBYAML:BOOL=${TARANTOOL_BUNDLED_LIBYAML} \
    -DWITH_SYSTEMD:BOOL=${TARANTOOL_WITH_SYSTEMD} \
    -DWITH_SYSVINIT:BOOL=${TARANTOOL_WITH_SYSVINIT} \
    -DWITH_LOGROTATE:BOOL=${TARANTOOL_WITH_LOGROTATE} \
    -DLUAJIT_ENABLE_GC64:BOOL=${LUAJIT_GC64} \
    .; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtimeDeps="$( \
    scanelf --needed --nobanner -F '%n #f' /usr/local/bin/tarantool \
      | tr ',' '\n' \
      | sort -u \
      | sed -e 's/^/so:\1/' \
      | xargs -r apk info --installed \
      | sort -u \
    )"; \
  apk add --no-cache --virtual .tarantool-runtime ${runtimeDeps}; \
  : "---------- Strip files ----------"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .tarantool-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["tarantool"]
