FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-v8-lib" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG V8_BRANCH="5.5-lkgr"
ARG V8_ARCH="x64.release"
ARG V8_SNAPSHOT="off"
ARG V8_LIB_DIR="/usr/local/v8"

RUN \
  apk add --update --virtual .v8-build-dependencies \
    bash \
    binutils-gold \
    curl \
    g++ \
    gcc \
    git \
    icu-dev \
    linux-headers \
    make \
    python \
    wget \
  && apk add \
    findutils \

  # depot tools
  && DEPOT_TOOLS_BRANCH="master" \
  && DEPOT_TOOLS_GIT="https://chromium.googlesource.com/chromium/tools/depot_tools.git" \
  && DEPOT_TOOLS_DIR="/tmp/depot_tools" \
  && git clone -b ${DEPOT_TOOLS_BRANCH} --depth 1 ${DEPOT_TOOLS_GIT} ${DEPOT_TOOLS_DIR} \
  && export PATH="${DEPOT_TOOLS_DIR}:${PATH}" \

  # V8
  && cd /tmp \
  && fetch v8 \
  && cd /tmp/v8 \
  && git checkout ${V8_BRANCH} \
  && gclient sync \

  # Does not work correctly
  && rm /tmp/v8/third_party/binutils/Linux_x64/Release/bin/* \
  && export PATH="/usr/x86_64-alpine-linux-musl/bin:${PATH}" \
  && export GYPFLAGS="-Dclang=0" \

  && make \
    ${V8_ARCH} \
    library=shared \
    snapshot=${V8_SNAPSHOT} \
    debugsymbols=off \
    werror=no \
    -j "$(getconf _NPROCESSORS_ONLN)" \

  # Shared libraries
  && V8_OUTPUT_DIR="/tmp/v8/out/${V8_ARCH}" \
  && mkdir -p ${V8_LIB_DIR}/include ${V8_LIB_DIR}/lib \
  && cp -R /tmp/v8/include/* ${V8_LIB_DIR}/include \
  && cp ${V8_OUTPUT_DIR}/lib.target/lib*.so ${V8_LIB_DIR}/lib \
  && echo -e "create ${V8_LIB_DIR}/lib/libv8_libbase.a\naddlib ${V8_OUTPUT_DIR}/obj.target/src/libv8_libbase.a\nsave\nend" | ar -M \
  && echo -e "create ${V8_LIB_DIR}/lib/libv8_libplatform.a\naddlib ${V8_OUTPUT_DIR}/obj.target/src/libv8_libplatform.a\nsave\nend" | ar -M \

  # Removing build dependencies, clean temporary files
  && apk del .v8-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*
