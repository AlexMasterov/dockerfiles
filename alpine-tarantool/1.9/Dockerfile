FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-tarantool:1.9" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_TAG=1.10.3-0-g0b7078a93
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=ON
ARG TARANTOOL_DIST=ON
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_BUNDLED_LUAJIT=ON
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
    binutils-dev \
    cmake \
    curl-dev \
    g++ \
    gcc \
    git \
    icu-dev \
    libressl-dev \
    make \
    ncurses-dev \
    perl \
    readline-dev \
    $(test "${TARANTOOL_BACKTRACE}" = 'ON' && echo libunwind-dev) \
    $(test "${TARANTOOL_BUNDLED_LIBYAML}" = 'OFF' && echo yaml-dev); \
  : "-- Tarantool"; \
  TARANTOOL_DIR="/tmp/tarantool"; \
  git clone https://github.com/tarantool/tarantool.git ${TARANTOOL_DIR}; \
  git -C ${TARANTOOL_DIR} checkout ${TARANTOOL_TAG}; \
  git -C ${TARANTOOL_DIR} submodule update --init --recursive --jobs $(getconf _NPROCESSORS_ONLN); \
  cd ${TARANTOOL_DIR}; \
  cmake \
    -DCMAKE_BUILD_TYPE=${TARANTOOL_BUILD_TYPE} \
    -DENABLE_BACKTRACE:BOOL=${TARANTOOL_BACKTRACE} \
    -DENABLE_AVX:BOOL=${TARANTOOL_AVX} \
    -DENABLE_DIST:BOOL=${TARANTOOL_DIST} \
    -DENABLE_BUNDLED_LUAJIT:BOOL=${TARANTOOL_BUNDLED_LUAJIT} \
    -DENABLE_BUNDLED_LIBYAML:BOOL=${TARANTOOL_BUNDLED_LIBYAML} \
    -DWITH_SYSTEMD:BOOL=${TARANTOOL_WITH_SYSTEMD} \
    -DWITH_SYSVINIT:BOOL=${TARANTOOL_WITH_SYSVINIT} \
    -DWITH_LOGROTATE:BOOL=${TARANTOOL_WITH_LOGROTATE} \
    -DLUAJIT_ENABLE_GC64:BOOL=${LUAJIT_GC64} \
    .; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/local/bin/tarantool \
      | tr ',' '\n' \
      | sort -u \
      | sed -e 's/^/so:\1/' \
    )"; \
  apk add --no-cache --virtual .tarantool-runtime ${runtime_deps}; \
  : "---------- Strip files ----------"; \
  find \
    /usr/bin \
    /usr/local/bin \
    -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .tarantool-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["tarantool"]
