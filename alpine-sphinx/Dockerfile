FROM alpine:3.4
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

ENV SPHINX_VERSION 2.2.11-release

RUN \
  addgroup -S sphinx \
  && adduser -D -S -h /var/cache/sphinx -s /sbin/nologin -G sphinx sphinx

RUN \
  apk add --update --no-cache --virtual .sphinx-build-dependencies \
    curl \
    expat-dev \
    g++ \
    gcc \
    libre2-dev \
    make \
    mariadb-dev \
    postgresql-dev \
    tar \
  && apk add --update-cache --virtual .sphinx-build-dependencies \
    --repository http://dl-3.alpinelinux.org/alpine/edge/community/ --allow-untrusted \
    snowball-dev \
  && apk add --no-cache --virtual .sphinx-runtime-dependencies \
    expat \
    libpq \
    libstdc++ \

  # Save MySQL client library
  && cp /usr/lib/libmysqlclient* /usr/local/lib \

  # Sphinx
  && SPHINX_FILENAME="sphinx-${SPHINX_VERSION}.tar.gz" \
  && SPHINX_SOURCE="http://sphinxsearch.com/files/${SPHINX_FILENAME}" \

  && curl -fSL ${SPHINX_SOURCE} -o /tmp/${SPHINX_FILENAME} \
  && tar xzf /tmp/${SPHINX_FILENAME} -C /tmp \
  && cd /tmp/sphinx-${SPHINX_VERSION} \

  # Patches
  && SPHINX_PATCH_DIR="/tmp/sphinx-patches/" \
  && mkdir -p ${SPHINX_PATCH_DIR} \

  && PAGESIZE_PATCH="http://bit.ly/2dtP10a" \
  && curl -fSL --connect-timeout 30 ${PAGESIZE_PATCH} -o ${SPHINX_PATCH_DIR}/patch1.patch \

  # Applying patches
  && patch -p1 -i ${SPHINX_PATCH_DIR}* \

  && ./configure \
    --prefix=/usr \
    --exec-prefix=/usr \
    --localstatedir=/var/lib/sphinx \
    --sysconfdir=/etc/sphinx \
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
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ENV TINI_VERSION v0.10.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["searchd"]
