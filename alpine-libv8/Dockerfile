FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-libv8" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG V8_VERSION=6.4.7
ARG V8_DIR=/usr/local/v8

ARG BUILD_COMMIT=f630878e35a30bb662c26ede03651446d3877cf0
ARG BUILDTOOLS_COMMIT=f6d165d9d842ddd29056c127a5f3a3c5d8e0d2e3
ARG ICU_COMMIT=21d33b1a09a77f033478ea4ffffb61e6970f83bd
ARG GTEST_COMMIT=6f8a66431cb592dad629028a50b3dd418a408c87
ARG TRACE_EVENT_COMMIT=abcc4153b783b5e2c2dafcfbf658017ecb56989a
ARG GYP_COMMIT=d61a9397e668fa9843c4aa7da9e79460fe590bfb
ARG CLANG_COMMIT=0235fed9a32f921614500213b8e3fc1a9f1f9bcb
ARG JINJA2_COMMIT=d34383206fa42d52faa10bb9931d6d538f3a57e0
ARG MARKUPSAFE_COMMIT=8f45f5cfa0009d2a70589bcda0349b8cb2b72783

ARG GN_SOURCE=https://www.dropbox.com/s/69lrj7y0yfyia8e/alpine-gn-ba97155.tar.gz
ARG V8_SOURCE=https://chromium.googlesource.com/v8/v8/+archive/${V8_VERSION}.tar.gz

ENV V8_VERSION=${V8_VERSION} \
    V8_DIR=${V8_DIR}

RUN set -x \
  && apk add --update --virtual .v8-build-dependencies \
    curl \
    g++ \
    gcc \
    glib-dev \
    icu-dev \
    linux-headers \
    make \
    ninja \
    python \
    tar \
    xz \
  && : "---------- V8 ----------" \
  && mkdir -p /tmp/v8 \
  && curl -fSL --connect-timeout 30 ${V8_SOURCE} | tar xmz -C /tmp/v8 \
  && : "---------- Dependencies ----------" \
  && DEPS=" \
    chromium/buildtools.git@${BUILDTOOLS_COMMIT}:buildtools; \
    chromium/src/build.git@${BUILD_COMMIT}:build; \
    chromium/src/base/trace_event/common.git@${TRACE_EVENT_COMMIT}:base/trace_event/common; \
    chromium/src/tools/clang.git@${CLANG_COMMIT}:tools/clang; \
    chromium/src/third_party/jinja2.git@${JINJA2_COMMIT}:third_party/jinja2; \
    chromium/src/third_party/markupsafe.git@${MARKUPSAFE_COMMIT}:third_party/markupsafe; \
    chromium/deps/icu.git@${ICU_COMMIT}:third_party/icu; \
    external/gyp.git@${GYP_COMMIT}:tools/gyp; \
    external/github.com/google/googletest.git@${GTEST_COMMIT}:testing/gtest \
  " \
  && while [ "${DEPS}" ]; do \
    dep="${DEPS%%;*}" \
    link="${dep%%:*}" \
    url="${link%%@*}" url="${url#"${url%%[![:space:]]*}"}" \
    hash="${link#*@}" \
    dir="${dep#*:}"; \
    [ -n "${dep}" ] \
      && dep_url="https://chromium.googlesource.com/${url}/+archive/${hash}.tar.gz" \
      && dep_dir="/tmp/v8/${dir}" \
      && mkdir -p ${dep_dir} \
      && curl -fSL --connect-timeout 30 ${dep_url} | tar xmz -C ${dep_dir} \
      & [ "${DEPS}" = "${dep}" ] && DEPS='' || DEPS="${DEPS#*;}"; \
    done; \
    wait \
  && : "---------- Downloads the current stable Linux sysroot ----------" \
  && /tmp/v8/build/linux/sysroot_scripts/install-sysroot.py \
    --arch=amd64 \
  && : "---------- Proper GN ----------" \
  && apk add --virtual .gn-runtime-dependencies \
    libevent \
    libexecinfo \
    libstdc++ \
  && curl -fSL --connect-timeout 30 ${GN_SOURCE} | tar xmz -C /tmp/v8/buildtools/linux64/ \
  && : "---------- Build instructions ----------" \
  && cd /tmp/v8 \
  && ./tools/dev/v8gen.py \
    x64.release \
    -- \
      binutils_path=\"/usr/bin\" \
      target_os=\"linux\" \
      target_cpu=\"x64\" \
      v8_target_cpu=\"x64\" \
      v8_use_external_startup_data=false \
      v8_enable_future=true \
      is_official_build=true \
      is_component_build=true \
      is_cfi=false \
      is_clang=false \
      is_debug=false \
      use_custom_libcxx=false \
      use_sysroot=false \
      use_gold=false \
      use_allocator=\"tcmalloc\" \
      use_allocator_shim=false \
      treat_warnings_as_errors=false \
      symbol_level=0 \
  && : "---------- Build ----------" \
  && ninja -C out.gn/x64.release/ d8 \
  && : "---------- Extract shared libraries ----------" \
  && mkdir -p ${V8_DIR}/include ${V8_DIR}/lib \
  && cp -R /tmp/v8/include/* ${V8_DIR}/include/ \
  && (cd /tmp/v8/out.gn/x64.release; \
      cp lib*.so icudtl.dat ${V8_DIR}/lib/) \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .v8-build-dependencies .gn-runtime-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*
