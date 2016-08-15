FROM alpine:3.4
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

ENV REDIS_VERSION 3.2.3

RUN \
  addgroup -S redis \
  && adduser -D -S -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN \
  apk add --update --no-cache --virtual .redis-build-dependencies \
    curl \
    gcc \
    linux-headers \
    make \
    musl-dev \
    tar \

  # Install Redis
  && REDIS_FILENAME="redis-${REDIS_VERSION}.tar.gz" \
  && REDIS_SOURCE="http://download.redis.io/releases/${REDIS_FILENAME}" \

  && curl -fSL ${REDIS_SOURCE} -o /tmp/${REDIS_FILENAME} \
  && tar xzf /tmp/${REDIS_FILENAME} -C /tmp \
  && cd /tmp/redis-${REDIS_VERSION} \

  && make -j "$(getconf _NPROCESSORS_ONLN)" \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .redis-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ENV TINI_VERSION v0.10.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["redis-server"]
