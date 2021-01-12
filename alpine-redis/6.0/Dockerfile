FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-redis:6.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG REDIS_VERSION=6.0.10

ARG REDIS_SERVER=YES
ARG REDIS_CLI=NO
ARG REDIS_BENCHMARK=NO
ARG REDIS_SSL=YES

ARG EXTRA_CFLAGS=-march=native

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S redis; \
  adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN set -eux; \
  if_option() { [ ${1} = ${2} ] && echo ${3:-} || echo ${4:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar xz -C /tmp; }; \
  apk add --no-cache --virtual .redis-build \
    g++ \
    gcc \
    linux-headers \
    make \
    pkgconfig \
    $(if_option ${REDIS_SSL} 'YES' openssl-dev); \
  : "-- Download Redis"; \
  load_archive "https://github.com/redis/redis/archive/${REDIS_VERSION}.tar.gz"; \
  cd /tmp/redis-${REDIS_VERSION}/src; \
  : "-- Patch 'src/config.c': disable Redis protected mode"; \
  sed -ri "s/(server.protected_mode), 1/\1, 0/" config.c; \
  : "-- Build Redis"; \
  make \
    $(if_option ${REDIS_SERVER}    'YES' redis-server) \
    $(if_option ${REDIS_CLI}       'YES' redis-cli) \
    $(if_option ${REDIS_BENCHMARK} 'YES' redis-benchmark) \
    BUILD_TLS=${REDIS_SSL} \
    EXTRA_CFLAGS=${EXTRA_CFLAGS} \
    -j $(getconf _NPROCESSORS_ONLN); \
  mv \
    $(if_option ${REDIS_SERVER}    'YES' redis-server) \
    $(if_option ${REDIS_CLI}       'YES' redis-cli) \
    $(if_option ${REDIS_BENCHMARK} 'YES' redis-benchmark) \
    /usr/local/bin; \
  : "-- Strip files"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .redis-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["redis-server"]
