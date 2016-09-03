FROM alpine:3.4
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

ENV MEMCACHED_VERSION 1.4.31

RUN \
  addgroup -S memcache \
  && adduser -D -S -h /var/cache/memcache -s /sbin/nologin -G memcache memcache

RUN \
  apk add --update --no-cache --virtual .memcached-build-dependencies \
    gcc \
    libevent-dev \
    make \
    tar \
    curl \
    g++ \
  && apk add --no-cache --virtual .memcached-runtime-dependencies \
    libevent \

  # Install Memcached
  && MEMCACHED_FILENAME="memcached-${MEMCACHED_VERSION}.tar.gz" \
  && MEMCACHED_SOURCE="https://memcached.org/files/${MEMCACHED_FILENAME}" \

  && curl -fSL ${MEMCACHED_SOURCE} -o /tmp/${MEMCACHED_FILENAME} \
  && tar xzf /tmp/${MEMCACHED_FILENAME} -C /tmp \
  && cd /tmp/memcached-${MEMCACHED_VERSION} \

  && ./configure \
  && make -j "$(getconf _NPROCESSORS_ONLN)" \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .memcached-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ENV TINI_VERSION v0.10.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["memcached"]
