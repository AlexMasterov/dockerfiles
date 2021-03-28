FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-memcached:1.6" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG MEMCACHED_VERSION=1.6.9

ARG MEMCACHED_TLS=YES
ARG MEMCACHED_EXTSTORE=YES
ARG MEMCACHED_CFLAGS="-march=native -O2"

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S memcache; \
  adduser -u 82 -S -D -h /var/cache/memcache -s /sbin/nologin -G memcache memcache

RUN set -eux; \
  if_yes() { [ ${1} = 'YES' ] && shift 1 && echo ${*:-}; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  \
  apk add --no-cache --virtual .memcached-build \
    g++ \
    gcc \
    libevent-dev \
    make \
    $(if_yes ${MEMCACHED_TLS} openssl-dev); \
  : "-- Memcached: download"; \
  load_archive "https://memcached.org/files/memcached-${MEMCACHED_VERSION}.tar.gz"; \
  cd /tmp/memcached-${MEMCACHED_VERSION}; \
  : "-- Memcached: build"; \
  ./configure \
    --disable-docs \
    --disable-coverage \
    --$(if_yes ${MEMCACHED_TLS}      enable || echo disable)-tls \
    --$(if_yes ${MEMCACHED_EXTSTORE} enable || echo disable)-extstore \
    CFLAGS="${MEMCACHED_CFLAGS:-}" \
    CPPFLAGS="${MEMCACHED_CFLAGS:-}"; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/local/bin/memcached \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .memcached-runtime ${runtime_deps}; \
  : "-- Strip files"; \
  find /usr/local/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .memcached-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["memcached"]
