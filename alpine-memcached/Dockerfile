FROM alpine:3.14 as build

LABEL repository.hub="alexmasterov/alpine-memcached" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 1.6.1
ARG MEMCACHED_VERSION=1.6.12

ARG MEMCACHED_TLS=YES
ARG MEMCACHED_EXTSTORE=YES
ARG MEMCACHED_SECCOMP=YES
ARG MEMCACHED_64BIT=NO

ARG MEMCACHED_CFLAGS="-O2"

ARG OUT_STRIP=YES

ARG USE_TZ=YES
ARG TZ="Europe/Moscow"

RUN set -eux; \
  as_ver() { set -- ${1//./ }; local y="00${2:-0}" z="00${3:-0}"; echo "${1}${y:(-3)}${z:(-3)}"; }; \
  compare_ver() { test $(as_ver ${1}) -${2} $(as_ver ${3}); }; \
  get_deps() { find ${*:-} -type f -and \( -executable -or -name '*.so*' \) -exec ldd '{}' ';' | sed -rn "s/\s*(.* => )?(.*) \(.*/\2/p" | sort -u; }; \
  if_memcached_ver() { if compare_ver ${MEMCACHED_VERSION} ${1} ${2}; then shift 2; echo "${*:-}"; fi; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo ${*:-}; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo ${*:-}; fi; }; \
  load_archive() { wget ${1} -T 30 -O - | tar ${2:-xmz} -C /tmp; }; \
  \
  apk add --no-cache --virtual .memcached-build \
    autoconf \
    automake \
    g++ \
    gcc \
    libevent-dev \
    libevent-static \
    make \
    $(if_memcached_ver ge 1.6.11 bsd-compat-headers) \
    $(if_yes ${MEMCACHED_SECCOMP} libseccomp-dev) \
    $(if_yes ${MEMCACHED_TLS} \
      openssl-dev \
      openssl-libs-static \
    ) \
    \
    upx \
    $(if_yes ${USE_TZ} tzdata) \
    ; \
  : "-- Memcached: download source"; \
  load_archive "https://memcached.org/files/memcached-${MEMCACHED_VERSION}.tar.gz"; \
  cd /tmp/memcached-${MEMCACHED_VERSION}; \
  : "-- Memcached: patch"; \
  sed -i "s/if test \"x\$ac_cv_search_pthread_create\".*/if false; then/" configure configure.ac; \
  if compare_ver ${MEMCACHED_VERSION} ge 1.6.11; then \
    : "-- Fix BSD header include (queue)"; \
    sed -ri "s/(#include) \"queue.h\"/\1 <sys\/queue.h>/" thread.c cache.h; \
  fi; \
  : "-- Memcached: build"; \
  if compare_ver ${MEMCACHED_VERSION} le 1.6.10; then \
    autoreconf; \
  fi; \
  ./configure \
    --enable-static \
    --disable-docs \
    --disable-coverage \
    --$(if_yes ${MEMCACHED_TLS}      enable)$(if_no ${MEMCACHED_TLS}      disable)-tls \
    --$(if_yes ${MEMCACHED_EXTSTORE} enable)$(if_no ${MEMCACHED_EXTSTORE} disable)-extstore \
    --$(if_yes ${MEMCACHED_SECCOMP}  enable)$(if_no ${MEMCACHED_SECCOMP}  disable)-seccomp \
    --$(if_yes ${MEMCACHED_64BIT}    enable)$(if_no ${MEMCACHED_64BIT}    disable)-64bit \
    CFLAGS="${MEMCACHED_CFLAGS:-}" \
    CPPFLAGS="${MEMCACHED_CFLAGS:-}"; \
  make memcached -j $(getconf _NPROCESSORS_ONLN); \
  : "-- Memcached: install"; \
  mv memcached /usr/local/bin; \
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

FROM busybox as runtime

COPY --from=build /out /

ENTRYPOINT ["/bin/memcached"]
