FROM golang:1.9-alpine

LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

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

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["/go/bin/MailHog"]
