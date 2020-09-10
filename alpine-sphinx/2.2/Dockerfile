FROM alpine:3.11

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
  apk add --no-cache --virtual .sphinx-build-dependencies \
    expat-dev \
    g++ \
    gcc \
    make \
    mariadb-connector-c-dev \
    postgresql-dev \
    re2-dev \
    snowball-dev; \
  apk add --virtual .sphinx-runtime-dependencies \
    expat \
    libpq \
    libstdc++ \
    mariadb-connector-c; \
  : "---------- Sphinx ----------"; \
  SPHINX_SOURCE="http://sphinxsearch.com/files/sphinx-${SPHINX_VERSION}.tar.gz"; \
  wget ${SPHINX_SOURCE} -T 30 -O - | tar xz -C /tmp; \
  cd /tmp/sphinx-${SPHINX_VERSION}; \
  : "---------- Patches ----------"; \
  PAGESIZE_PATCH="http://git.alpinelinux.org/cgit/aports/plain/community/sphinx/sphinx-pagesize.patch"; \
  wget ${PAGESIZE_PATCH} -T 30 -O - | patch -p1 -u; \
  : "---------- Build ----------"; \
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
  : "---------- Removing build dependencies, clean temporary files ----------"; \
  apk del .sphinx-build-dependencies; \
  rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

CMD ["searchd"]
