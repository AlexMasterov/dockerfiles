FROM alpine:edge

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-redis:4.0" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

ARG REDIS_VERSION=4.0-rc3
ARG REDIS_MALLOC=jemalloc

RUN \
  apk add --update \
    tini \

  && addgroup -g 82 -S redis \
  && adduser -u 82 -S -D -h /var/cache/redis -s /sbin/nologin -G redis redis

RUN \
  apk add --virtual .redis-build-dependencies \
    curl \
    gcc \
    linux-headers \
    make \
    musl-dev \

  && REDIS_SOURCE="https://github.com/antirez/redis/archive/${REDIS_VERSION}.tar.gz" \
  && curl -fSL --connect-timeout 30 ${REDIS_SOURCE} | tar xz -C /tmp \
  && cd /tmp/redis-${REDIS_VERSION} \

  && make \
    MALLOC=${REDIS_MALLOC} \
    CFLAGS="-march=native" \
    -j $(getconf _NPROCESSORS_ONLN) \
  && make install \

  # Removing build dependencies, clean temporary files
  && apk del .redis-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["redis-server"]
