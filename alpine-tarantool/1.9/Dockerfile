FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-tarantool:1.9" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_TAG=1.9.0
# [ RelWithDebInfo | Release ]
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=ON
ARG TARANTOOL_DIST=ON
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_BUNDLED_LIBYAML=OFF
ARG TARANTOOL_WITH_SYSTEMD=OFF
ARG TARANTOOL_WITH_SYSVINIT=ON
ARG TARANTOOL_WITH_LOGROTATE=ON

ARG LUAJIT_GC64=ON

RUN set -x \
  && apk add --update \
    tini \
  && addgroup -g 82 -S tarantool \
  && adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN set -x \
  && apk add --virtual .tarantool-build-dependencies \
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
    $(test "${TARANTOOL_BACKTRACE}" = 'ON' && echo 'libunwind-dev') \
    $(test "${TARANTOOL_BUNDLED_LIBYAML}" = 'OFF' && echo 'yaml-dev') \
  && : "---------- Tarantool ----------" \
  && TARANTOOL_DIR="/tmp/tarantool" \
  && git clone -o ${TARANTOOL_TAG} https://github.com/tarantool/tarantool.git ${TARANTOOL_DIR} \
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
  && runtimeDeps="$( \
    scanelf --needed --nobanner --recursive /usr/local/bin/tarantool \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
  )" \
  && apk add --virtual .tarantool-runtime-dependencies \
    ${runtimeDeps} \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .tarantool-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["tarantool"]
