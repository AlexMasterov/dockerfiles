FROM alpine:3.13 as build

LABEL repository.hub="alexmasterov/alpine-redis" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 3.2.0
ARG REDIS_VERSION=6.2.2

ARG REDIS_CLI=NO
ARG REDIS_SERVER=YES
ARG REDIS_BENCHMARK=NO
ARG REDIS_TLS=YES

ARG REDIS_EXTRA_CFLAGS

ARG JEMALLOC_LG_PAGE
ARG JEMALLOC_LG_HUGEPAGE
ARG JEMALLOC_LG_QUANTUM=3
ARG JEMALLOC_EXTRA_CONFIGURE="--disable-stats --disable-lazy-lock"

ARG OUT_STRIP=YES

ARG USE_TZ=YES
ARG TZ="Europe/Moscow"

RUN set -eux; \
  as_ver() { set -- ${1//./ }; local y="00${2:-0}" z="00${3:-0}"; echo "${1}${y:(-3)}${z:(-3)}"; }; \
  compare_ver() { test $(as_ver ${1}) -${2} $(as_ver ${3}); }; \
  if_redis_ver() { if compare_ver ${REDIS_VERSION} ${1} ${2}; then shift 2; echo "${*:-}"; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo "${*:-}"; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  \
  apk add --no-cache --virtual .redis-build \
    gcc \
    make \
    musl-dev \
    pkgconf \
    upx \
    $(if_redis_ver le 6.0.11 linux-headers) \
    $(if_redis_ver ge 6.0.0 $(if_yes ${REDIS_TLS} \
      openssl-dev \
      openssl-libs-static \
    )) \
    \
    $(if_yes ${USE_TZ} tzdata) \
    ; \
  : "-- Redis: download source"; \
  load_archive "https://download.redis.io/releases/redis-${REDIS_VERSION}.tar.gz"; \
  cd /tmp/redis-${REDIS_VERSION}/src; \
  \
  : "-- jemalloc: configure"; \
  jemalloc_extra_configure=" \
    ${JEMALLOC_LG_QUANTUM:+--with-lg-quantum=${JEMALLOC_LG_QUANTUM}} \
    ${JEMALLOC_LG_HUGEPAGE:+--with-lg-hugepage=${JEMALLOC_LG_HUGEPAGE}} \
    ${JEMALLOC_LG_PAGE:+--with-lg-page=${JEMALLOC_LG_PAGE}} \
    ${JEMALLOC_EXTRA_CONFIGURE:-} \
  "; \
  sed -ri "s/(pthread_create_fptr) = dlsym.*;/\1 = pthread_create;/" ../deps/jemalloc/src/background_thread.c; \
  sed -ri \
    -e "s/\s?--with-lg-(page|hugepage|quantum)=\d+//g" \
    -e "s/cd jemalloc && .\/configure /&${jemalloc_extra_configure// [[:space:]]/}/" \
    ../deps/Makefile; \
  \
  : "-- Redis: disable protected mode"; \
  sed -i "s/(server.protected_mode), 1/\1, 0/" config.c; \
  : "-- Redis: build"; \
  build_targets="\
    $(if_yes ${REDIS_CLI}       redis-cli) \
    $(if_yes ${REDIS_SERVER}    redis-server) \
    $(if_yes ${REDIS_BENCHMARK} redis-benchmark) \
  "; \
  make \
    ${build_targets} \
    $(if_redis_ver ge 6.0.0 $(if_yes ${REDIS_TLS} BUILD_TLS=yes)) \
    LDFLAGS="-static" \
    EXTRA_CFLAGS="${REDIS_EXTRA_CFLAGS:-}" \
    -j $(getconf _NPROCESSORS_ONLN) \
    ; \
  : "-- Redis: install"; \
  mv ${build_targets} /usr/local/bin; \
  \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    : "-- Out: strip executables"; \
    find /usr/local -type f -executable \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  upx --no-color --no-progress --best --lzma \
    /usr/local/bin/*; \
  \
  mkdir -p /out; \
  cp --recursive --dereference --link \
    /usr/local/bin/ \
    /out; \
  \
  if [ ${USE_TZ} = 'YES' ]; then \
    cp /usr/share/zoneinfo/${TZ} /etc/localtime; \
    echo "${TZ}" > /etc/timezone; \
    \
    cp --recursive --parents --dereference --link \
      /etc/timezone \
      /etc/localtime \
      /out; \
  fi; 

FROM scratch as runtime

COPY --from=build /out /

ENTRYPOINT ["/bin/redis-server"]
