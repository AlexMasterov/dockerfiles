FROM golang:1.8-alpine

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

RUN \
  apk add --update \
    bzr \
    tini \
  && apk add --virtual .mailhog-build-dependencies \
    git \

  && go get github.com/mailhog/MailHog \

  # Removing build dependencies, clean temporary files
  && apk del .mailhog-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ENTRYPOINT ["tini", "--"]
CMD ["/go/bin/MailHog"]
