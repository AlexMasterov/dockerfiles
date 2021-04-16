FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-tarantool" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG TARANTOOL_TAG=2.6.2-0-g34d504d
ARG TARANTOOL_BUILD_TYPE=RelWithDebInfo
ARG TARANTOOL_AVX=YES
ARG TARANTOOL_BACKTRACE=YES
ARG TARANTOOL_BUNDLED_LIBCURL=NO
ARG TARANTOOL_BUNDLED_LIBCURL_ARES=NO
ARG TARANTOOL_BUNDLED_LIBYAML=NO
ARG TARANTOOL_BUNDLED_ZSTD=NO
ARG TARANTOOL_DIST=NO

ARG LUAJIT_GC64=YES

ARG USE_GPERFTOOLS=NO
ARG GPERFTOOLS_VERSION=2.9.1
ARG GPERFTOOLS_CPU_PROFILER=YES
ARG GPERFTOOLS_HEAP_PROFILER=YES
ARG GPERFTOOLS_HEAP_CHECKER=YES

ARG USE_PPROF=NO
ARG PPROF_REV

ARG USE_TZ=YES
ARG LIB_SSL=OPENSSL

ARG OUT_STRIP=YES

RUN set -eux; \
  get_deps() { scanelf --nobanner --needed --recursive -F '%n#p' ${*:-} | tr ',' '\n' | sort -u; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo "${*:-}"; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo "${*:-}"; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  to_lowercase() { echo "${1:-}" | tr '[:upper:]' '[:lower:]'; }; \
  \
  addgroup -g 82 -S tarantool; \
  adduser -u 82 -S -D -h /var/cache/tarantool -s /sbin/nologin -G tarantool tarantool; \
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
    $(to_lowercase ${LIB_SSL:-OPENSSL})-dev \
    $(if_yes ${TARANTOOL_BACKTRACE}        libunwind-dev) \
    $(if_yes ${TARANTOOL_BUNDLED_LIBCURL}  autoconf) \
    $(if_no  ${TARANTOOL_BUNDLED_LIBCURL}  curl-dev) \
    $(if_no  ${TARANTOOL_BUNDLED_LIBYAML}  yaml-dev) \
    $(if_no  ${TARANTOOL_BUNDLED_ZSTD}     zstd-dev) \
    \
    $(if_yes ${USE_GPERFTOOLS} linux-headers) \
    $(if_yes ${USE_PPROF}      go) \
    ; \
  if [ ${USE_TZ} = 'YES' ]; then \
    apk add --no-cache --virtual .tarantool-tz tzdata; \
  fi; \
  : "-- Tarantool: download source"; \
  git clone \
    --no-progress \
    --no-checkout \
    --filter=blob:none \
    https://github.com/tarantool/tarantool.git /tmp/tarantool; \
  git -C /tmp/tarantool checkout ${TARANTOOL_TAG}; \
  \
  cd /tmp/tarantool; \
  : "-- Patch: CMakeLists.txt (remove update_submodules)"; \
  sed -i '/update_submodules()/d' CMakeLists.txt; \
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
  \
  : "-- Patch: lib/{msgpuck,small}/CMakeLists.txt (remove test)"; \
  sed -i '/add_subdirectory(test)/d' CMakeLists.txt; \
  sed -i "s/if (NOT \${PROJECT_SOURCE_DIR}.*)/if (TRUE)/" src/lib/msgpuck/CMakeLists.txt; \
  sed -ri "/(enable_testing\(\)|add_subdirectory\((test|perf)\))/d" src/lib/small/CMakeLists.txt; \
  \
  : "-- Tarantool: build"; \
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
  \
  if [ ${USE_GPERFTOOLS} = 'YES' ]; then \
  : "-- gperftools: download source"; \
    load_archive "https://github.com/gperftools/gperftools/archive/refs/tags/gperftools-${GPERFTOOLS_VERSION}.tar.gz"; \
    cd /tmp/gperftools-gperftools-${GPERFTOOLS_VERSION}; \
    : "-- gperftools: build"; \
    cmake \
      -DBUILD_TESTING:BOOL=NO \
      -DGPERFTOOLS_BUILD_DEBUGALLOC:BOOL=NO \
      -Dgperftools_build_benchmark:BOOL=NO \
      -DDEFAULT_BUILD_CPU_PROFILER:BOOL=${GPERFTOOLS_CPU_PROFILER} \
      -DDEFAULT_BUILD_HEAP_PROFILER:BOOL=${GPERFTOOLS_HEAP_PROFILER} \
      -DDEFAULT_BUILD_HEAP_CHECKER:BOOL=${GPERFTOOLS_HEAP_CHECKER} \
      .; \
    make \
      -j $(getconf _NPROCESSORS_ONLN) \
      profiler; \
    mv libprofiler.so* /usr/local/lib; \
  fi; \
  if [ ${USE_PPROF} = 'YES' ]; then \
    export \
      GO111MODULE=on \
      GOPATH=/tmp/go \
      GOCACHE=/tmp/go/.cache; \
    : "-- pprof: build"; \
    go get github.com/google/pprof@${PPROF_REV:-master}; \
    mv /tmp/go/bin/pprof /usr/local/bin; \
  fi; \
  \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    : "-- Out: strip executables"; \
    find /usr/local -type f -executable \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  runtime_deps=$( \
    scanelf --nobanner --needed --recursive -F '%n#p' /usr/local/bin/ /usr/local/lib/ \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    ); \
  \
  apk add --no-cache --virtual .tarantool-runtime \
    $(get_deps /usr/local/bin /usr/local/lib | sed "s/^/so:&/"); \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .tarantool-build; \
  rm -rf /tmp/* /var/tmp/*

USER tarantool

CMD ["tarantool"]
