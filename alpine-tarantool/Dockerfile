FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-tarantool:1.7" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG TARANTOOL_VERSION=1.7.4-0-g0ed962422
# [ RelWithDebInfo | Release ]
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=YES
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_DIST=NO
ARG TARANTOOL_BUNDLED_LIBYAML=NO

RUN \
  apk add --update \
    tini \

  && addgroup -g 82 -S tarantool \
  && adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN \
  apk add --no-cache --virtual .tarantool-build-dependencies \
    binutils-dev \
    cmake \
    curl \
    g++ \
    gcc \
    git \
    libressl-dev \
    make \
    ncurses-dev \
    perl \
    readline-dev \
    yaml-dev \
    zlib-dev \
  && apk add --virtual .tarantool-runtime-dependencies \
    libgomp \
    libstdc++ \
    ncurses \
    readline \
    yaml \

  # Tarantool
  && TARANTOOL_DIR="/tmp/tarantool" \
  && git clone https://github.com/tarantool/tarantool.git ${TARANTOOL_DIR} \
  && git -C ${TARANTOOL_DIR} checkout ${TARANTOOL_VERSION} \
  && git -C ${TARANTOOL_DIR} submodule update \
    --init --recursive --jobs "$(getconf _NPROCESSORS_ONLN)" \
  && cd ${TARANTOOL_DIR} \

  && cmake \
    -DCMAKE_BUILD_TYPE=${TARANTOOL_BUILD_TYPE} \
    -DENABLE_BACKTRACE:BOOL=${TARANTOOL_BACKTRACE} \
    -DENABLE_AVX:BOOL=${TARANTOOL_AVX} \
    -DENABLE_DIST:BOOL=${TARANTOOL_DIST} \
    -DENABLE_BUNDLED_LIBYAML:BOOL=${TARANTOOL_BUNDLED_LIBYAML} \
    . \
  && make -j "$(getconf _NPROCESSORS_ONLN)" \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .tarantool-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["tarantool"]
