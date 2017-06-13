# dockerfiles

Common docker containers, used for development.

**Images**

- [alpine-libv8](#alpine-libv8)
- alpine-mailhog
- alpine-memcached
- alpine-nginx-tarantool
- alpine-nginx
- alpine-php
- alpine-redis
- alpine-sphinx
- alpine-tarantool

### alpine-libv8
```sh
docker build --build-arg V8_VERSION=6.1.147 --build-arg V8_DIR=/usr/local/v8 .
```
#### docker-compose
```yaml
build:
  context: ./alpine-libv8
  args:
    - V8_VERSION=6.1.147
    - V8_DIR=/usr/local/v8
```
#### Extract
```sh
docker run -v $(pwd):/archive alpine_libv8 sh -c 'tar cvzf /archive/libv8-${V8_VERSION}.tar.gz -C ${V8_DIR} .'
```
