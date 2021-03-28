FROM golang:1.16-alpine3.13 as build

LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

RUN set -eux; \
  apk add --no-cache \
    git; \
  : "-- MailHog: download"; \
  go get github.com/mailhog/MailHog;

FROM alpine:3.13
COPY --from=build /go/bin/MailHog /usr/local/bin

RUN set -eux; \
  addgroup -g 82 -S mailhog; \
  adduser -u 82 -S -D -h /var/cache/mailhog -s /sbin/nologin -G mailhog mailhog;

USER mailhog
EXPOSE 8025/tcp

CMD ["MailHog"]
