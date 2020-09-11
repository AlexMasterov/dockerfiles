FROM golang:1.15-alpine

LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

RUN set -eux; \
  apk add --no-cache --virtual .mailhog-build \
    git; \
  : "-- Download MailHog"; \
  go get github.com/mailhog/MailHog; \
  : "-- Removing build dependencies, clean temporary files"; \
  apk del .mailhog-build; \
  rm -rf /tmp/* /var/tmp/*

WORKDIR /go/bin

CMD ["MailHog"]
