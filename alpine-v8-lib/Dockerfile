FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-v8-lib" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG V8_BRANCH="5.7.455"
ARG V8_LIB_DIR="/usr/local/v8"

COPY patch/*.patch /tmp/gn_patch/

RUN \
  apk add --update --virtual .v8-build-dependencies \
    bash \
    binutils-gold \
    g++ \
    gcc \
    git \
    glib-dev \
    icu-dev \
    linux-headers \
    make \
    ninja \
    python \
    tar \
  && apk add --virtual .gn-build-dependencies \
    libevent-dev \
    libexecinfo-dev \
    patch \

  # GN
  && GN_DIR="/tmp/gn" \

  && CHROMIUM_SOURCE="https://chromium.googlesource.com/chromium" \
  && git clone ${CHROMIUM_SOURCE}/src/tools/gn ${GN_DIR}/tools/gn \
  && git -C ${GN_DIR}/tools/gn checkout "5d5a530c87977ec853ff1ed1c14de1ad8e1a0de2" \
  && git clone ${CHROMIUM_SOURCE}/src/base ${GN_DIR}/base \
  && git -C ${GN_DIR}/base checkout "6320afce757cb8f9893c9004ae1abafd494f4d21" \
  && git clone --depth 1 ${CHROMIUM_SOURCE}/src/build ${GN_DIR}/build \
  && git clone --depth 1 ${CHROMIUM_SOURCE}/testing/gtest ${GN_DIR}/testing/gtest \

  && cd ${GN_DIR} \

  # Patches
  && for patch in /tmp/gn_patch/*.patch; do patch -p0 -i ${patch}; done \

  # Bootstrapping GN
  && ./tools/gn/bootstrap/bootstrap.py --no-rebuild \

  # depot tools
  && DEPOT_TOOLS_DIR="/tmp/depot_tools" \
  && DEPOT_TOOLS_GIT="https://chromium.googlesource.com/chromium/tools/depot_tools.git" \
  && git clone --depth 1 ${DEPOT_TOOLS_GIT} ${DEPOT_TOOLS_DIR} \
  && export PATH="${PATH}:${DEPOT_TOOLS_DIR}" \

  # V8
  && cd /tmp \
  && fetch v8 \
  && cd /tmp/v8 \
  && git checkout ${V8_BRANCH} \
  && gclient sync \

  # Does not work correctly
  && cp -f ${GN_DIR}/out/Release/gn /tmp/v8/buildtools/linux64/gn \

  && ./tools/dev/v8gen.py \
    x64.release \
    -- \
      binutils_path=\"/usr/bin\" \
      target_os=\"linux\" \
      target_cpu=\"x64\" \
      v8_target_cpu=\"x64\" \
      is_component_build=true \
      is_clang=false \
      is_debug=false \
      use_sysroot=false \
      use_gold=false \
      use_allocator=\"tcmalloc\" \
      use_experimental_allocator_shim=false \
      enable_nacl=false \
      treat_warnings_as_errors=false \
      symbol_level=0 \

  # Ninja turtles GO!
  && ninja -C out.gn/x64.release/ \

  # Shared libraries
  && mkdir -p ${V8_LIB_DIR}/include ${V8_LIB_DIR}/lib \
  && cp -R /tmp/v8/include/* ${V8_LIB_DIR}/include/ \
  && cp /tmp/v8/out.gn/x64.release/lib*.so /tmp/v8/out.gn/x64.release/*_blob.bin ${V8_LIB_DIR}/lib/ \

  # Removing build dependencies, clean temporary files
  && apk del .v8-build-dependencies .gn-build-dependencies \
  && rm -rf /tmp/* /var/tmp/* /var/cache/apk/*
