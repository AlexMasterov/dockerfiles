FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-tarantool:1.7" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_VERSION=1.7.4-329-g2e4b6fddf
# [ RelWithDebInfo | Release ]
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=ON
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_DIST=ON
ARG TARANTOOL_BUNDLED_LIBYAML=OFF
ARG TARANTOOL_WITH_SYSTEMD=OFF
ARG TARANTOOL_WITH_SYSVINIT=ON
ARG TARANTOOL_WITH_LOGROTATE=ON

RUN \
  apk add --update \
    tini \

  && addgroup -g 82 -S tarantool \
  && adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN \
  if [[ $TARANTOOL_BACKTRACE = "ON" ]]; then \
    buildDeps="libunwind-dev"; \
    runtimeDeps="libunwind"; \
  fi \
  && \
  if [[ $TARANTOOL_BUNDLED_LIBYAML = "OFF" ]]; then \
    buildDeps="${buildDeps} yaml-dev"; \
    runtimeDeps="${runtimeDeps} yaml"; \
  fi \

  && apk add --virtual .tarantool-build-dependencies \
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
    ${buildDeps} \
  && apk add --virtual .tarantool-runtime-dependencies \
    curl \
    libgomp \
    libstdc++ \
    ncurses \
    readline \
    ${runtimeDeps} \

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
    . \
  && make -j $(getconf _NPROCESSORS_ONLN) \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .tarantool-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["tarantool"]
