FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-tarantool:2.5" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_TAG=2.5.1-1-g635f6e5
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_AVX=YES
ARG TARANTOOL_BACKTRACE=NO
ARG TARANTOOL_BUNDLED_LIBCURL=NO
ARG TARANTOOL_BUNDLED_LIBCURL_ARES=NO
ARG TARANTOOL_BUNDLED_LIBYAML=NO
ARG TARANTOOL_BUNDLED_ZSTD=NO
ARG TARANTOOL_DIST=NO

ARG LUAJIT_GC64=YES

ARG SSL_LIB=OPENSSL

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S tarantool; \
  adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool

RUN set -eux; \
  if_yes() { [ ${1} = 'YES' ] && shift 1 && echo ${*:-}; }; \
  if_no() { [ ${1} = 'NO' ] && shift 1 && echo ${*:-}; }; \
  to_lowercase() { echo ${1:-} | tr '[:upper:]' '[:lower:]'; }; \
  \
  apk add --no-cache --virtual .tarantool-build \
    cmake \
    g++ \
    gcc \
    git \
    icu-dev \
    make \
    ncurses-dev \
    readline-dev \
    vim \
    $(to_lowercase ${LIB_SSL})-dev \
    $(if_yes ${TARANTOOL_BACKTRACE}       libunwind-dev) \
    $(if_no ${TARANTOOL_BUNDLED_LIBYAML}  yaml-dev) \
    $(if_no ${TARANTOOL_BUNDLED_ZSTD}     zstd-dev) \
    $(if_yes ${TARANTOOL_BUNDLED_LIBCURL} autoconf || echo curl-dev) \
    ; \
  : "-- Download Tarantool source"; \
  git clone --no-progress https://github.com/tarantool/tarantool.git /tmp/tarantool; \
  cd /tmp/tarantool; \
  git checkout ${TARANTOOL_TAG}; \
  : "-- Patch: CMakeLists.txt (remove update_submodules)"; \
  sed -i 's/update_submodules()//g' CMakeLists.txt; \
  : "-- Git: update submodules"; \
  git \
    -c submodule."test-run".update=none \
    $(if_no ${TARANTOOL_BUNDLED_ZSTD}         -c submodule."third_party/zstd".update=none) \
    $(if_no ${TARANTOOL_BUNDLED_LIBYAML}      -c submodule."third_party/libyaml".update=none) \
    $(if_no ${TARANTOOL_BUNDLED_LIBCURL}      -c submodule."third_party/curl".update=none) \
    $(if_no ${TARANTOOL_BUNDLED_LIBCURL_ARES} -c submodule."third_party/c-ares".update=none) \
    submodule update \
      --init \
      --recursive \
      --jobs $(getconf _NPROCESSORS_ONLN); \
  cmake \
    -DCMAKE_BUILD_TYPE=${TARANTOOL_BUILD_TYPE:-RelWithDebInfo} \
    -DENABLE_BACKTRACE:BOOL=${TARANTOOL_BACKTRACE} \
    -DENABLE_DIST:BOOL=${TARANTOOL_DIST} \
    -DENABLE_AVX:BOOL=${TARANTOOL_AVX} \
    -DENABLE_BUNDLED_LIBCURL:BOOL=${TARANTOOL_BUNDLED_LIBCURL} \
    -DENABLE_BUNDLED_LIBYAML:BOOL=${TARANTOOL_BUNDLED_LIBYAML} \
    -DENABLE_BUNDLED_ZSTD:BOOL=${TARANTOOL_BUNDLED_ZSTD} \
    -DLUAJIT_ENABLE_GC64:BOOL=${LUAJIT_GC64} \
    .; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/local/bin/tarantool \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .tarantool-runtime ${runtime_deps}; \
  : "-- Strip files"; \
  find /usr/lib/* -type f -name '*.a' -delete; \
  find \
    /usr/bin \
    /usr/sbin \
    /usr/local/bin \
    -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .tarantool-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["tarantool"]
