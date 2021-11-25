FROM alpine:3.15 as build

LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

ARG MAILHOG_VERSION=1.0.1

RUN set -eux; \
  apk add --no-cache \
    gcc \
    git \
    go \
    upx \
    ; \
  : "-- MailHog: download source & build"; \
  go install \
    -gcflags=all='-l -B' \
    -ldflags '-s -w -linkmode external -extldflags -static' \
    github.com/mailhog/MailHog@v${MAILHOG_VERSION}; \
  \
  upx --no-color --no-progress --best --lzma \
    /root/go/bin/MailHog

FROM scratch as runtime

COPY --from=build /root/go/bin/MailHog /MailHog

ENTRYPOINT ["/MailHog"]
