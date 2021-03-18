FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-redis:5.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG REDIS_VERSION=5.0.12

ARG REDIS_SERVER=YES
ARG REDIS_CLI=NO
ARG REDIS_BENCHMARK=NO
ARG REDIS_EXTRA_CFLAGS="-march=native"

ARG JEMALLOC_LG_PAGE=
ARG JEMALLOC_LG_HUGEPAGE=
ARG JEMALLOC_LG_QUANTUM=3
ARG JEMALLOC_EXTRA_CFLAGS="-march=native"

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S redis; \
  adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN set -eux; \
  if_gt() { [ ${1} -gt ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  if_eq() { [ ${1} = ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xz} -C /tmp; }; \
  apk add --no-cache --virtual .redis-build \
    g++ \
    gcc \
    linux-headers \
    make; \
  : "-- Download Redis"; \
  load_archive "https://github.com/redis/redis/archive/${REDIS_VERSION}.tar.gz"; \
  cd /tmp/redis-${REDIS_VERSION}/src; \
  : "-- Patch 'deps/Makefile': jemalloc configure"; \
  JEMALLOC_EXTRA_CONFIGURE="\
    $(if_gt ${JEMALLOC_LG_PAGE}     0 --with-lg-page=${JEMALLOC_LG_PAGE}) \
    $(if_gt ${JEMALLOC_LG_HUGEPAGE} 0 --with-lg-hugepage=${JEMALLOC_LG_HUGEPAGE}) \
    $(if_gt ${JEMALLOC_LG_QUANTUM}  0 --with-lg-quantum=${JEMALLOC_LG_QUANTUM}) \
  "; \
  sed -ir \
    -e "$([ -n "${JEMALLOC_EXTRA_CFLAGS}" ] && echo "s/JEMALLOC_CFLAGS= /&${JEMALLOC_EXTRA_CFLAGS} /")" \
    -e "s/\s?--with-lg-(page|hugepage|quantum)=\d+//g" \
    -e "s/cd jemalloc && .\/configure /&${JEMALLOC_EXTRA_CONFIGURE} /" \
    ../deps/Makefile; \
  : "-- Patch 'src/config.c': disable Redis protected mode"; \
  sed -i 's/^(#define CONFIG_DEFAULT_PROTECTED_MODE).+/\1 0/' server.h; \
  : "-- Build Redis"; \
  make \
    $(if_eq ${REDIS_SERVER}    'YES' redis-server) \
    $(if_eq ${REDIS_CLI}       'YES' redis-cli) \
    $(if_eq ${REDIS_BENCHMARK} 'YES' redis-benchmark) \
    EXTRA_CFLAGS="${REDIS_EXTRA_CFLAGS}" \
    -j $(getconf _NPROCESSORS_ONLN); \
  mv \
    $(if_eq ${REDIS_SERVER}    'YES' redis-server) \
    $(if_eq ${REDIS_CLI}       'YES' redis-cli) \
    $(if_eq ${REDIS_BENCHMARK} 'YES' redis-benchmark) \
    /usr/local/bin; \
  : "-- Strip files"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .redis-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["redis-server"]
