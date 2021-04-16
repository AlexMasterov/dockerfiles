FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-redis" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 3.2.0
ARG REDIS_VERSION=6.2.1

ARG REDIS_CLI=NO
ARG REDIS_SERVER=YES
ARG REDIS_BENCHMARK=NO
ARG REDIS_TLS=YES
ARG REDIS_EXTRA_CFLAGS

ARG JEMALLOC_LG_HUGEPAGE
ARG JEMALLOC_LG_PAGE
ARG JEMALLOC_LG_QUANTUM=3
ARG JEMALLOC_EXTRA_CFLAGS

ARG USE_TZ=YES
ARG LIB_SSL=OPENSSL

ARG OUT_STRIP=YES

RUN set -eux; \
  as_ver() { set -- ${1//./ }; local y="00${2:-0}" z="00${3:-0}"; echo "${1}${y:(-3)}${z:(-3)}"; }; \
  compare_ver() { test $(as_ver ${1}) -${2} $(as_ver ${3}); }; \
  get_deps() { scanelf --nobanner --needed --recursive -F '%n#p' ${*:-} | tr ',' '\n' | sort -u; }; \
  if_redis_ver() { if compare_ver ${REDIS_VERSION} ${1} ${2}; then shift 2; echo "${*:-}"; fi; }; \
  if_set() { if [ -n ${1} ]; then shift 1; echo "${*:-}"; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo "${*:-}"; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  to_lowercase() { echo ${1:-} | tr '[:upper:]' '[:lower:]'; }; \
  \
  addgroup -g 82 -S redis; \
  adduser -u 82 -S -D -h /var/local/cache/redis -s /sbin/nologin -G redis redis; \
  \
  apk add --no-cache --virtual .redis-build \
    g++ \
    gcc \
    linux-headers \
    make \
    pkgconf \
    $(if_redis_ver ge 6.0.0 \
      $(if_yes ${REDIS_TLS} $(to_lowercase ${LIB_SSL})-dev) \
    ) \
    ; \
  if [ ${USE_TZ} = 'YES' ]; then \
    apk add --no-cache --virtual .redis-tz tzdata; \
  fi; \
  \
  : "-- Redis: download"; \
  load_archive "https://download.redis.io/releases/redis-${REDIS_VERSION}.tar.gz"; \
  cd /tmp/redis-${REDIS_VERSION}/src; \
  \
  if compare_ver ${REDIS_VERSION} ge 3.2.0; then \
    : "-- Patch: disable protected mode"; \
    sed -i "s/(server.protected_mode), 1/\1, 0/" config.c; \
  fi; \
  : "-- Patch: jemalloc configure"; \
  JEMALLOC_EXTRA_CONFIGURE="\
    $(if_set ${JEMALLOC_LG_PAGE}     --with-lg-page=${JEMALLOC_LG_PAGE}) \
    $(if_set ${JEMALLOC_LG_HUGEPAGE} --with-lg-hugepage=${JEMALLOC_LG_HUGEPAGE}) \
    $(if_set ${JEMALLOC_LG_QUANTUM}  --with-lg-quantum=${JEMALLOC_LG_QUANTUM}) \
  "; \
  sed -ri \
    -e "$(if_set ${JEMALLOC_EXTRA_CFLAGS} "s/JEMALLOC_CFLAGS= /&${JEMALLOC_EXTRA_CFLAGS} /")" \
    -e "s/\s?--with-lg-(page|hugepage|quantum)=\d+//g" \
    -e "s/cd jemalloc && .\/configure /&${JEMALLOC_EXTRA_CONFIGURE// [[:space:]]/}/" \
    ../deps/Makefile; \
  \
  : "-- Redis: build"; \
  build_targets="\
    $(if_yes ${REDIS_CLI}       redis-cli) \
    $(if_yes ${REDIS_SERVER}    redis-server) \
    $(if_yes ${REDIS_BENCHMARK} redis-benchmark) \
  "; \
  make \
    ${build_targets} \
    EXTRA_CFLAGS="${REDIS_EXTRA_CFLAGS:-}" \
    $(if_redis_ver ge 6.0.0 \
      $(if_yes ${REDIS_TLS} BUILD_TLS=yes) \
    ) \
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
  apk add --no-cache --virtual .redis-runtime \
    $(get_deps /usr/local/bin /usr/local/lib | sed "s/^/so:&/"); \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .redis-build; \
  rm -rf /tmp/* /var/tmp/*

USER redis

CMD ["redis-server"]
