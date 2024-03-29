FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-sphinx:2.2" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG SPHINX_VERSION=2.2.11-release

RUN set -eux; \
  apk add --no-cache \
    tzdata; \
  addgroup -g 82 -S sphinx; \
  adduser -u 82 -S -D -h /var/cache/sphinx -s /sbin/nologin -G sphinx sphinx

RUN set -eux; \
  load_archive() { wget ${1} -T 30 -O - | tar xz -C /tmp; }; \
  apk add --no-cache --virtual .sphinx-build \
    expat-dev \
    g++ \
    gcc \
    make \
    mariadb-connector-c-dev \
    patch \
    postgresql-dev \
    re2-dev \
    snowball-dev; \
  : "-- Download Sphinx"; \
  load_archive "https://sphinxsearch.com/files/sphinx-${SPHINX_VERSION}.tar.gz"; \
  cd /tmp/sphinx-${SPHINX_VERSION}; \
  : "-- Patch 'src/sphinx.cpp': pagesize"; \
  PAGESIZE_PATCH="https://git.alpinelinux.org/cgit/aports/plain/community/sphinx/sphinx-pagesize.patch"; \
  wget ${PAGESIZE_PATCH} -T 30 -O - | patch -p1 -u; \
  : "-- Build Sphinx"; \
  ./configure \
    --prefix=/usr \
    --exec-prefix=/usr \
    --sysconfdir=/etc/sphinx \
    --localstatedir=/var/lib/sphinx \
    --enable-id64 \
    --with-iconv \
    --with-libexpat \
    --with-libstemmer \
    --with-mysql \
    --with-pgsql \
    --with-re2; \
  make -j $(getconf _NPROCESSORS_ONLN); \
  make install; \
  runtime_deps="$( \
    scanelf --needed --nobanner -F '%n#p' /usr/bin/searchd \
      | tr ',' '\n' \
      | sort -u \
      | sed 's/^/so:&/' \
    )"; \
  apk add --no-cache --virtual .sphinx-runtime ${runtime_deps}; \
  : "-- Strip files"; \
  find /usr/bin -type f -perm +0111 -exec strip --strip-unneeded '{}' + || true; \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .sphinx-build; \
  rm -rf /tmp/* /var/tmp/*

CMD ["searchd"]
