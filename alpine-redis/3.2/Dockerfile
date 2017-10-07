FROM alpine:edge

LABEL repository.hub="alexmasterov/alpine-redis:3.2" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG REDIS_VERSION=3.2.11
ARG REDIS_MALLOC=jemalloc

RUN set -x \
  && apk add --update \
    tini \
  && addgroup -g 82 -S redis \
  && adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN set -x \
  && apk add --virtual .redis-build-dependencies \
    curl \
    gcc \
    linux-headers \
    make \
    musl-dev \
  && : "---------- Redis ----------" \
  && REDIS_SOURCE="http://download.redis.io/releases/redis-${REDIS_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${REDIS_SOURCE} | tar xz -C /tmp \
  && cd /tmp/redis-${REDIS_VERSION} \
  && : "---------- Build ----------" \
  && make \
    MALLOC=${REDIS_MALLOC} \
    CFLAGS="-march=native" \
    -j $(getconf _NPROCESSORS_ONLN) \
  && make install \
  && : "---------- Removing build dependencies, clean temporary files ----------" \
  && apk del .redis-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["redis-server"]
