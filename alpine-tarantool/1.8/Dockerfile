FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-tarantool:1.8" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_VERSION=1.8.1-415-ge3d2485c7
# [ RelWithDebInfo | Release ]
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=ON
ARG TARANTOOL_DIST=ON
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_BUNDLED_LIBYAML=OFF
ARG TARANTOOL_WITH_SYSTEMD=OFF
ARG TARANTOOL_WITH_SYSVINIT=ON
ARG TARANTOOL_WITH_LOGROTATE=ON

ARG LUAJIT_GC64=OFF

RUN \
  apk add --update \
    tini \

  && addgroup -g 82 -S tarantool \
  && adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN \
  [ $TARANTOOL_BACKTRACE = "ON" ] && \
    buildDeps="libunwind-dev" \
    runtimeDeps="libunwind" \
  || true \
  [ $TARANTOOL_BUNDLED_LIBYAML = "OFF" ] && \
    buildDeps="${buildDeps} yaml-dev" \
    runtimeDeps="${runtimeDeps} yaml" \
  || true \

  && apk add --virtual .tarantool-build-dependencies \
    ${buildDeps} \
    binutils-dev \
    cmake \
    curl-dev \
    g++ \
    gcc \
    git \
    libressl-dev \
    make \
    ncurses-dev \
    perl \
    readline-dev \
    zlib-dev \
  && apk add --virtual .tarantool-runtime-dependencies \
    ${runtimeDeps} \
    curl \
    libgomp \
    libstdc++ \
    ncurses \
    readline \

  # Tarantool
  && TARANTOOL_DIR="/tmp/tarantool" \
  && git clone https://github.com/tarantool/tarantool.git ${TARANTOOL_DIR} \
  && git -C ${TARANTOOL_DIR} checkout ${TARANTOOL_VERSION} \
  && git -C ${TARANTOOL_DIR} submodule update \
    --init --recursive --jobs $(getconf _NPROCESSORS_ONLN) \
  && cd ${TARANTOOL_DIR} \

  && cmake \
    -DCMAKE_BUILD_TYPE=${TARANTOOL_BUILD_TYPE} \
    -DENABLE_BACKTRACE:BOOL=${TARANTOOL_BACKTRACE} \
    -DENABLE_AVX:BOOL=${TARANTOOL_AVX} \
    -DENABLE_DIST:BOOL=${TARANTOOL_DIST} \
    -DENABLE_BUNDLED_LIBYAML:BOOL=${TARANTOOL_BUNDLED_LIBYAML} \
    -DWITH_SYSTEMD:BOOL=${TARANTOOL_WITH_SYSTEMD} \
    -DWITH_SYSVINIT:BOOL=${TARANTOOL_WITH_SYSVINIT} \
    -DWITH_LOGROTATE:BOOL=${TARANTOOL_WITH_LOGROTATE} \
    -DLUAJIT_ENABLE_GC64:BOOL=${LUAJIT_GC64} \
    . \
  && make -j $(getconf _NPROCESSORS_ONLN) \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .tarantool-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["tarantool"]
