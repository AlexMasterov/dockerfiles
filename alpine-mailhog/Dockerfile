FROM golang:1.7.4-alpine

MAINTAINER Alex Masterov <alex.masterow@gmail.com>
LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles"

RUN \
  apk add --update --virtual .mailhog-build-dependencies \
    git \
  && apk add \
    bzr \

  && go get github.com/mailhog/MailHog \

  # Removing build dependencies, clean temporary files
  && apk del .mailhog-build-dependencies \
  && rm -rf /var/cache/apk/* /var/tmp/* /tmp/*

ARG TINI_VERSION=v0.14.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["/go/bin/MailHog"]
