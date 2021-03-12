FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-memcached:1.5" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG MEMCACHED_VERSION=1.5.22

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S memcache; \
  adduser -u 82 -S -D -h /var/cache/memcache -s /sbin/nologin -G memcache memcache

RUN set -eux; \
  apk add --no-cache --virtual .memcached-build \
    g++ \
    gcc \
    libevent-dev \
    make \
    openssl-dev; \
  : "---------- Download Memcached ----------"; \
  MEMCACHED_SOURCE="https://memcached.org/files/memcached-${MEMCACHED_VERSION}.tar.gz"; \
  wget ${MEMCACHED_SOURCE} -T 30 -O - | tar xz -C /tmp; \
  cd /tmp/memcached-${MEMCACHED_VERSION}; \
  : "---------- Build Memcached----------"; \
  ./configure \
    CFLAGS="-O2 -march=native" \
    --enable-tls; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtimeDeps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/local/bin/memcached \
      | tr ',' '\n' \
      | sort -u \
      | sed -e 's/^/so:\1/' \
    )"; \
  apk add --no-cache --virtual .memcached-runtime ${runtimeDeps}; \
  : "-- Strip files"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .memcached-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["memcached"]
