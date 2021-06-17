FROM alpine:3.14 as build

LABEL repository.hub="alexmasterov/alpine-mailhog" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

RUN set -eux; \
  apk add --no-cache \
    gcc \
    git \
    go \
    upx \
    ; \
  : "-- MailHog: download source & build"; \
  go get \
    -gcflags=all='-l -B' \
    -ldflags '-s -w -linkmode external -extldflags -static' \
    github.com/mailhog/MailHog; \
  \
  upx --no-color --no-progress --best --lzma \
    /root/go/bin/MailHog

FROM scratch as runtime

COPY --from=build /root/go/bin/MailHog /MailHog

ENTRYPOINT ["/MailHog"]
