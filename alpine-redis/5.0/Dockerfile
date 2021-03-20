FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-redis:5.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG REDIS_VERSION=5.0.12

ARG REDIS_BENCHMARK=NO
ARG REDIS_CLI=NO
ARG REDIS_SERVER=YES
ARG REDIS_EXTRA_CFLAGS="-march=native"

ARG JEMALLOC_LG_HUGEPAGE=
ARG JEMALLOC_LG_PAGE=
ARG JEMALLOC_LG_QUANTUM=3
ARG JEMALLOC_EXTRA_CFLAGS="-march=native"

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S redis; \
  adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN set -eux; \
  if_eq() { [ ${1} = ${2} ] && shift 2 && echo ${*:-} || exit 0; }; \
  if_not_empty() { [ -n "${1}" ] && shift 1 && echo ${*:-} || exit 0; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xz} -C /tmp; }; \
  apk add --no-cache --virtual .redis-build \
    g++ \
    gcc \
    linux-headers \
    make; \
  : "-- Redis: download"; \
  load_archive "https://github.com/redis/redis/archive/${REDIS_VERSION}.tar.gz"; \
  cd /tmp/redis-${REDIS_VERSION}/src; \
  : "-- Redis: patch 'deps/Makefile' (jemalloc configure)"; \
  JEMALLOC_EXTRA_CONFIGURE="\
    `if_not_empty ${JEMALLOC_LG_PAGE}     --with-lg-page=${JEMALLOC_LG_PAGE}` \
    `if_not_empty ${JEMALLOC_LG_HUGEPAGE} --with-lg-hugepage=${JEMALLOC_LG_HUGEPAGE}` \
    `if_not_empty ${JEMALLOC_LG_QUANTUM}  --with-lg-quantum=${JEMALLOC_LG_QUANTUM}` \
  "; \
  sed -ir \
    -e "`if_not_empty ${JEMALLOC_EXTRA_CFLAGS} "s/JEMALLOC_CFLAGS= /&${JEMALLOC_EXTRA_CFLAGS} /"`" \
    -e "s/\s?--with-lg-(page|hugepage|quantum)=\d+//g" \
    -e "s/cd jemalloc && .\/configure /&${JEMALLOC_EXTRA_CONFIGURE// [[:space:]]/}/" \
    ../deps/Makefile; \
  : "-- Redis: patch 'src/config.c' (disable protected mode)"; \
  sed -ir "s/^(#define CONFIG_DEFAULT_PROTECTED_MODE).+/\1 0/" server.h; \
  : "-- Redis: make"; \
  BUILD_TARGETS="\
    `if_eq ${REDIS_SERVER}    'YES' redis-server` \
    `if_eq ${REDIS_CLI}       'YES' redis-cli` \
    `if_eq ${REDIS_BENCHMARK} 'YES' redis-benchmark` \
  "; \
  make \
    ${BUILD_TARGETS} \
    EXTRA_CFLAGS="${REDIS_EXTRA_CFLAGS}" \
    -j $(getconf _NPROCESSORS_ONLN); \
  : "-- Redis: make install"; \
  mv ${BUILD_TARGETS} /usr/local/bin; \
  : "-- Strip files"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .redis-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["redis-server"]
