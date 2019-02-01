FROM alpine:3.9

LABEL repository.hub="alexmasterov/alpine-libv8:7.4" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG V8_VERSION=7.4.51
ARG V8_DIR=/usr/local/v8

ARG BUILD_COMMIT=c1ab94d375f10578b3d207eca8fe4fb35274efb7
ARG BUILDTOOLS_COMMIT=6fbda1b24c1893a893b17aa219b765b9e7c801d8
ARG ICU_COMMIT=07e7295d964399ee7bee16a3ac7ca5a053b2cf0a
ARG GTEST_COMMIT=879ac092fde0a19e1b3a61b2546b2a422b1528bc
ARG TRACE_EVENT_COMMIT=e31a1706337ccb9a658b37d29a018c81695c6518
ARG CLANG_COMMIT=3114fbc11f9644c54dd0a4cdbfa867bac50ff983
ARG JINJA2_COMMIT=b41863e42637544c2941b574c7877d3e1f663e25
ARG MARKUPSAFE_COMMIT=8f45f5cfa0009d2a70589bcda0349b8cb2b72783
ARG CATAPULT_COMMIT=b6cc5a6baf93cfa6feeb240eea75c454506b0c3c

ARG GN_SOURCE=https://www.dropbox.com/s/3ublwqh4h9dit9t/alpine-gn-80e00be.tar.gz
ARG V8_SOURCE=https://chromium.googlesource.com/v8/v8/+archive/${V8_VERSION}.tar.gz

ENV V8_VERSION=${V8_VERSION} \
    V8_DIR=${V8_DIR}

RUN set -x \
  && apk add --update --virtual .v8-build-dependencies \
    at-spi2-core-dev \
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
    external/github.com/google/googletest.git@${GTEST_COMMIT}:third_party/googletest/src; \
    catapult.git@${CATAPULT_COMMIT}:third_party/catapult \
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
  && /tmp/v8/build/linux/sysroot_scripts/install-sysroot.py --arch=amd64 \
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
      is_official_build=true \
      is_component_build=true \
      is_cfi=false \
      is_clang=false \
      use_custom_libcxx=false \
      use_sysroot=false \
      use_gold=false \
      use_allocator_shim=false \
      treat_warnings_as_errors=false \
      symbol_level=0 \
  && : "---------- Build ----------" \
  && ninja d8 -C out.gn/x64.release/ -j $(getconf _NPROCESSORS_ONLN) \
  && : "---------- Extract shared libraries ----------" \
  && mkdir -p ${V8_DIR}/include ${V8_DIR}/lib \
  && cp -R /tmp/v8/include/* ${V8_DIR}/include/ \
  && (cd /tmp/v8/out.gn/x64.release; \
      cp lib*.so icudtl.dat ${V8_DIR}/lib/) \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .v8-build-dependencies .gn-runtime-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*
