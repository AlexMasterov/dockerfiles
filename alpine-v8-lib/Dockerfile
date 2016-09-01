FROM alpine:3.4
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

ENV V8_BRANCH       5.4.455
ENV V8_ARCHITECTURE native
ENV V8_OUTPUT_DIR   /usr/local/v8

RUN \
  apk add --update --no-cache --virtual .v8-build-dependencies \
    bash \
    binutils-gold \
    curl \
    g++ \
    gcc \
    git \
    icu-dev \
    linux-headers \
    make \
    perl \
    python \
    tar \
  && apk add --no-cache \
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

  # See: https://bugs.chromium.org/p/v8/issues/detail?id=5249
  && export GYPFLAGS="-Dwerror= " \

  && export GYPFLAGS="-Dv8_use_external_startup_data=0 -Dclang=0 ${GYPFLAGS}" \
  && make \
    ${V8_ARCHITECTURE} \
    library=shared \
    snapshot=on \
    -j "$(getconf _NPROCESSORS_ONLN)" \

  && mkdir -p ${V8_OUTPUT_DIR}/include ${V8_OUTPUT_DIR}/lib \
  && cp -R /tmp/v8/include/* ${V8_OUTPUT_DIR}/include \
  && cp /tmp/v8/out/${V8_ARCHITECTURE}/lib.target/lib* ${V8_OUTPUT_DIR}/lib \
  && echo -e \
    "create ${V8_OUTPUT_DIR}/lib/libv8_libplatform.a\naddlib /tmp/v8/out/${V8_ARCHITECTURE}/obj.target/src/libv8_libplatform.a\nsave\nend" \
    | ar -M \

  # Removing build dependencies, clean temporary files
  && apk del .v8-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*
