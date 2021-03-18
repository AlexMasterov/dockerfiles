FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-redis:4.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG REDIS_VERSION=4.0.14

ARG REDIS_SERVER=YES
ARG REDIS_CLI=NO
ARG REDIS_BENCHMARK=NO
ARG REDIS_EXTRA_CFLAGS="-march=native"

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S redis; \
  adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN set -eux; \
  if_eq() { [ ${1} = ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xz} -C /tmp; }; \
  apk add --no-cache --virtual .redis-build \
    gcc \
    linux-headers \
    make \
    musl-dev; \
  : "-- Download Redis"; \
  load_archive "https://github.com/redis/redis/archive/${REDIS_VERSION}.tar.gz"; \
  cd /tmp/redis-${REDIS_VERSION}/src; \
  : "-- Patch 'src/config.c': disable Redis protected mode"; \
  sed -ri 's/^(#define CONFIG_DEFAULT_PROTECTED_MODE).+/\1 0/' server.h; \
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
