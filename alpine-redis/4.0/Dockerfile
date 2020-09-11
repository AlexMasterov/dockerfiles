FROM alpine:3.12

LABEL repository.hub="alexmasterov/alpine-redis:4.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG REDIS_VERSION=4.0.14
ARG REDIS_MALLOC=jemalloc

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S redis; \
  adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN set -eux; \
  apk add --no-cache --virtual .redis-build \
    gcc \
    linux-headers \
    make \
    musl-dev; \
  : "-- Download Redis"; \
  REDIS_SOURCE="https://github.com/antirez/redis/archive/${REDIS_VERSION}.tar.gz"; \
  wget ${REDIS_SOURCE} -T 30 -O - | tar xz -C /tmp; \
  cd /tmp/redis-${REDIS_VERSION}; \
  : "-- Patch 'src/Makefile': remove deduplicate 'redis-server' copies"; \
  INSTALL_REGEXP='^\s+?\$\(REDIS_INSTALL\) \$\(REDIS_CHECK_(RDB|AOF)_NAME\) \$\(INSTALL_BIN\)'; \
  LN_REGEXP='(^\s+?@ln -sf \$\(REDIS_SERVER_NAME\) \$\(INSTALL_BIN\)\/).+$'; \
  LN_REPLACE_REGEXP='\1$(REDIS_SENTINEL_NAME)\n\1$(REDIS_CHECK_RDB_NAME)\n\1$(REDIS_CHECK_AOF_NAME)'; \
  sed -ri "/${INSTALL_REGEXP}/d" src/Makefile; \
  sed -ri "s/${LN_REGEXP}/${LN_REPLACE_REGEXP}/" src/Makefile; \
  : "-- Patch 'src/config.c': disable Redis protected mode"; \
  sed -ri 's/^(#define CONFIG_DEFAULT_PROTECTED_MODE).+/\1 0/' src/server.h; \
  : "-- Build Redis"; \
  make \
    MALLOC=${REDIS_MALLOC} \
    CFLAGS="-march=native" \
    -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  : "-- Strip files"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Removing build dependencies, clean temporary files"; \
  apk del .redis-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["redis-server"]
