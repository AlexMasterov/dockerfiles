FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-libv8" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG V8_VERSION=6.0.75
ARG V8_DIR=/usr/local/v8

ENV V8_VERSION=${V8_VERSION} \
    V8_DIR=${V8_DIR}

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
    wget \

  # depot tools
  && DEPOT_TOOLS_DIR="/tmp/depot_tools" \
  && DEPOT_TOOLS_GIT="https://chromium.googlesource.com/chromium/tools/depot_tools.git" \
  && git clone --depth 1 ${DEPOT_TOOLS_GIT} ${DEPOT_TOOLS_DIR} \
  && export PATH="${PATH}:${DEPOT_TOOLS_DIR}" \

  # V8
  && cd /tmp \
  && fetch v8 \
  && cd /tmp/v8 \
  && gclient sync \
    -r ${V8_VERSION} \
    -j $(getconf _NPROCESSORS_ONLN) \

  # Proper GN
  && apk add --virtual .gn-runtime-dependencies \
    libevent \
    libexecinfo \
    libstdc++ \

  && GN_SOURCE="https://www.dropbox.com/s/69lrj7y0yfyia8e/alpine-gn-ba97155.tar.gz" \
  && wget --no-check-certificate -O - ${GN_SOURCE} | tar xz -C /tmp/v8/buildtools/linux64/ \

  && ./tools/dev/v8gen.py \
    x64.release \
    -- \
      binutils_path=\"/usr/bin\" \
      target_os=\"linux\" \
      target_cpu=\"x64\" \
      v8_target_cpu=\"x64\" \
      v8_use_external_startup_data=false \
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

  # Build
  && ninja -C out.gn/x64.release/ d8 \

  # Shared libraries
  && mkdir -p ${V8_DIR}/include ${V8_DIR}/lib \
  && cp -R /tmp/v8/include/* ${V8_DIR}/include/ \
  && (cd /tmp/v8/out.gn/x64.release; \
      cp lib*.so icudtl.dat ${V8_DIR}/lib/) \

  # Removing build dependencies, clean temporary files
  && apk del .v8-build-dependencies .gn-runtime-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*
