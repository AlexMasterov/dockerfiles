FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-libv8" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG V8_VERSION=6.3.138
ARG V8_DIR=/usr/local/v8

ARG BUILD_COMMIT=48a2b7b39debc7c77c868c9ddb0a360af1ebc367
ARG BUILDTOOLS_COMMIT=5af0a3a8b89827a8634132080a39ab4b63dee489
ARG ICU_COMMIT=08cb956852a5ccdba7f9c941728bb833529ba3c6
ARG GTEST_COMMIT=6f8a66431cb592dad629028a50b3dd418a408c87
ARG TRACE_EVENT_COMMIT=65d1d42a5df6c0a563a6fdfa58a135679185e5d9
ARG GYP_COMMIT=d61a9397e668fa9843c4aa7da9e79460fe590bfb
ARG CLANG_COMMIT=40f69660bf3cd407e72b8ae240fdd6c513dddbfe
ARG JINJA2_COMMIT=d34383206fa42d52faa10bb9931d6d538f3a57e0
ARG MARKUPSAFE_COMMIT=8f45f5cfa0009d2a70589bcda0349b8cb2b72783

ARG GN_SOURCE=https://www.dropbox.com/s/69lrj7y0yfyia8e/alpine-gn-ba97155.tar.gz

ENV V8_VERSION=${V8_VERSION} \
    V8_DIR=${V8_DIR}

RUN \
  apk add --update --virtual .v8-build-dependencies \
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
    xz \

  && CHROMIUM_SOURCE="https://chromium.googlesource.com" \
  && V8_SOURCE="https://chromium.googlesource.com/v8/v8" \
  && V8_SOURCE_DIR="/tmp/v8" \

  # V8
  && git clone -o ${V8_VERSION} --depth 1 ${V8_SOURCE} ${V8_SOURCE_DIR} \

  # Dependencies
  && BUILD_DIR="${V8_SOURCE_DIR}/build" \
  && git clone ${CHROMIUM_SOURCE}/chromium/src/build.git ${BUILD_DIR} \
  && git -C ${BUILD_DIR} checkout ${BUILD_COMMIT} \

  && BUILDTOOLS_DIR="${V8_SOURCE_DIR}/buildtools" \
  && git clone ${CHROMIUM_SOURCE}/chromium/buildtools.git ${BUILDTOOLS_DIR} \
  && git -C ${BUILDTOOLS_DIR} checkout ${BUILDTOOLS_COMMIT} \

  && TRACE_EVENT_DIR="${V8_SOURCE_DIR}/base/trace_event/common" \
  && git clone ${CHROMIUM_SOURCE}/chromium/src/base/trace_event/common.git ${TRACE_EVENT_DIR} \
  && git -C ${TRACE_EVENT_DIR} checkout ${TRACE_EVENT_COMMIT} \

  && GYP_DIR="${V8_SOURCE_DIR}/tools/gyp" \
  && git clone ${CHROMIUM_SOURCE}/external/gyp.git ${GYP_DIR} \
  && git -C ${GYP_DIR} checkout ${GYP_COMMIT} \

  && CLANG_DIR="${V8_SOURCE_DIR}/tools/clang" \
  && git clone ${CHROMIUM_SOURCE}/chromium/src/tools/clang.git ${CLANG_DIR} \
  && git -C ${CLANG_DIR} checkout ${CLANG_COMMIT} \

  && ICU_DIR="${V8_SOURCE_DIR}/third_party/icu" \
  && git clone ${CHROMIUM_SOURCE}/chromium/deps/icu.git ${ICU_DIR} \
  && git -C ${ICU_DIR} checkout ${ICU_COMMIT} \

  && JINJA2_DIR="${V8_SOURCE_DIR}/third_party/jinja2" \
  && git clone ${CHROMIUM_SOURCE}/chromium/src/third_party/jinja2.git ${JINJA2_DIR} \
  && git -C ${JINJA2_DIR} checkout ${JINJA2_COMMIT} \

  && MARKUPSAFE_DIR="${V8_SOURCE_DIR}/third_party/markupsafe" \
  && git clone ${CHROMIUM_SOURCE}/chromium/src/third_party/markupsafe.git ${MARKUPSAFE_DIR} \
  && git -C ${MARKUPSAFE_DIR} checkout ${MARKUPSAFE_COMMIT} \

  && GTEST_DIR="${V8_SOURCE_DIR}/testing/gtest" \
  && git clone ${CHROMIUM_SOURCE}/external/github.com/google/googletest.git ${GTEST_DIR} \
  && git -C ${GTEST_DIR} checkout ${GTEST_COMMIT} \

  # Downloads the current stable Linux sysroot
  && ${V8_SOURCE_DIR}/build/linux/sysroot_scripts/install-sysroot.py \
    --arch=amd64 \

  # Proper GN
  && apk add --virtual .gn-runtime-dependencies \
    libevent \
    libexecinfo \
    libstdc++ \

  && wget -O - ${GN_SOURCE} | tar xz -C /tmp/v8/buildtools/linux64/ \

  && cd ${V8_SOURCE_DIR} \
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
      is_clang=false \
      is_debug=false \
      use_custom_libcxx=false \
      use_sysroot=false \
      use_gold=false \
      use_allocator=\"tcmalloc\" \
      use_allocator_shim=false \
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
