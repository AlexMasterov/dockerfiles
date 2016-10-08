FROM golang:1.7.1-alpine
MAINTAINER Alex Masterov <alex.masterow@gmail.com>

RUN \
  apk add --update --virtual .mailhog-build-dependencies \
    git \
  && apk add \
    bzr \

  && go get github.com/mailhog/MailHog \

  # Removing build dependencies, clean temporary files
  && apk del .mailhog-build-dependencies \
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

ENV TINI_VERSION v0.10.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /sbin/tini
RUN chmod +x /sbin/tini

ENTRYPOINT ["tini", "--"]
CMD ["/go/bin/MailHog"]
