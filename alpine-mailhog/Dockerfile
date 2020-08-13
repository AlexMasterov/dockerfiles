FROM golang:1.15-alpine

LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

RUN set -eux; \
  apk add --update \
    bzr; \
  apk add --virtual .mailhog-build-dependencies \
    git; \
  : "---------- MailHog ----------"; \
  go get github.com/mailhog/MailHog; \
  : "---------- Removing build dependencies, clean temporary files ----------"; \
  apk del --no-network .mailhog-build-dependencies; \
  rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

CMD ["/go/bin/MailHog"]
