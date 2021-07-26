FROM alpine:3.14

LABEL repository.hub="alexmasterov/alpine-memcached" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 1.6.0
ARG MEMCACHED_VERSION=1.6.10

ARG MEMCACHED_TLS=YES
ARG MEMCACHED_EXTSTORE=YES
ARG MEMCACHED_SECCOMP=NO
ARG MEMCACHED_64BIT=NO
ARG MEMCACHED_CFLAGS="-O2"

ARG USE_TZ=YES
ARG LIB_SSL=OPENSSL

ARG OUT_STRIP=YES

RUN set -eux; \
  get_deps() { find ${*:-} -type f -and \( -executable -or -name '*.so*' \) -exec ldd '{}' ';' | sed -rn "s/\s*(.* => )?(.*) \(.*/\2/p" | sort -u; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo ${*:-}; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo ${*:-}; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  to_lowercase() { echo ${1:-} | tr '[:upper:]' '[:lower:]'; }; \
  \
  apk add --no-cache --virtual .memcached-build \
    autoconf \
    g++ \
    gcc \
    libevent-dev \
    make \
    $(if_yes ${MEMCACHED_TLS} $(to_lowercase ${LIB_SSL:-OPENSSL})-dev) \
    ; \
  if [ ${USE_TZ} = 'YES' ]; then \
    apk add --no-cache --virtual .memcached-tz tzdata; \
  fi; \
  : "-- Memcached: download source"; \
  load_archive "https://memcached.org/files/memcached-${MEMCACHED_VERSION}.tar.gz"; \
  cd /tmp/memcached-${MEMCACHED_VERSION}; \
  : "-- Memcached: build"; \
  ./configure \
    --disable-docs \
    --disable-coverage \
    --$(if_yes ${MEMCACHED_TLS}      enable)$(if_no ${MEMCACHED_TLS}      disable)-tls \
    --$(if_yes ${MEMCACHED_EXTSTORE} enable)$(if_no ${MEMCACHED_EXTSTORE} disable)-extstore \
    --$(if_yes ${MEMCACHED_SECCOMP}  enable)$(if_no ${MEMCACHED_SECCOMP}  disable)-seccomp \
    --$(if_yes ${MEMCACHED_64BIT}    enable)$(if_no ${MEMCACHED_64BIT}    disable)-64bit \
    CFLAGS="${MEMCACHED_CFLAGS:-}" \
    CPPFLAGS="${MEMCACHED_CFLAGS:-}"; \
  make memcached -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    : "-- Out: strip executables"; \
    find /usr/local -type f -executable \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  runtime_deps=$(get_deps /usr/local/bin /usr/local/lib | sed -rn "s/\/usr\/lib\/(.*)/so:\1/p"); \
  apk add --no-cache --virtual .memcached-runtime ${runtime_deps}; \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .memcached-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["memcached"]
