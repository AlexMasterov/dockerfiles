FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-sphinx" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG SPHINX_VERSION=2.2.11-release

RUN \
  addgroup -S sphinx \
  && adduser -D -S -h /var/cache/sphinx -s /sbin/nologin -G sphinx sphinx

RUN \
  apk add --update --virtual .sphinx-build-dependencies \
    curl \
    expat-dev \
    g++ \
    gcc \
    libre2-dev \
    make \
    mariadb-dev \
    postgresql-dev \
    snowball-dev \
  && apk add --virtual .sphinx-runtime-dependencies \
    expat \
    libpq \
    libstdc++ \

  # Save MySQL client library
  && cp /usr/lib/libmysqlclient* /usr/local/lib \

  # Sphinx
  && SPHINX_SOURCE="http://sphinxsearch.com/files/sphinx-${SPHINX_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${SPHINX_SOURCE} | tar xz -C /tmp \
  && cd /tmp/sphinx-${SPHINX_VERSION} \

  # Patches
  && PAGESIZE_PATCH="http://git.alpinelinux.org/cgit/aports/plain/community/sphinx/sphinx-pagesize.patch" \
  && curl -fSL --connect-timeout 30 ${PAGESIZE_PATCH} | patch -p1 -u \

  && ./configure \
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
    --with-re2 \
  && make -j "$(getconf _NPROCESSORS_ONLN)" \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .sphinx-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ARG TINI_VERSION=v0.14.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["searchd"]
