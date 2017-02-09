FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-tarantool:1.7" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG TARANTOOL_BRANCH=1.7
# [ RelWithDebInfo | Release ]
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_BACKTRACE=YES
ARG TARANTOOL_AVX=OFF
ARG TARANTOOL_DIST=NO
ARG TARANTOOL_BUNDLED_LIBYAML=NO

RUN \
  addgroup -S tarantool \
  && adduser -D -S -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN \
  apk add --update --virtual .tarantool-build-dependencies \
    cmake \
    curl \
    g++ \
    gcc \
    git \
    libressl-dev \
    make \
    perl \
    readline-dev \
    yaml-dev \
    zlib-dev \
  && apk add --virtual .tarantool-runtime-dependencies \
    libgomp \
    libstdc++ \
    readline \
    yaml \

  # Termcap
  && TERMCAP_VERSION="1.3.1" \
  && TERMCAP_SOURCE="https://ftp.gnu.org/pub/gnu/termcap/termcap-${TERMCAP_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${TERMCAP_SOURCE} | tar xz -C /tmp \
  && cd /tmp/termcap-${TERMCAP_VERSION} \

  && ./configure \
  && make \
  && make install \

  # Tarantool
  && TARANTOOL_GIT="https://github.com/tarantool/tarantool.git" \
  && TARANTOOL_DIR="/tmp/tarantool" \
  && git clone -b ${TARANTOOL_BRANCH} ${TARANTOOL_GIT} ${TARANTOOL_DIR} \
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

ARG TINI_VERSION=v0.14.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["tarantool"]
