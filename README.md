# dockerfiles

Common Dockerfiles, used for _development_.

- [alpine-v8](#alpine-v8)

#### alpine-v8

```sh
docker build --build-arg V8_VERSION=9.2.24 --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
> V8_VERSION >= 6.1.211

##### FAQ

<details>
<summary>How do I get an archive with V8?</summary>

```sh
docker run --rm -v "$(pwd)/:/archive" v8 sh -c 'tar cvzf /archive/libv8-${V8_VERSION}.tar.gz *'
```
</details>

<details>
<summary>How do I get shell (d8)?</summary>

```sh
docker build \
  --build-arg V8_VERSION=9.2.24 \
  --build-arg V8_SHELL=YES \
  --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
```sh
docker run --rm -it v8 d8
```
</details>

<details>
<summary>How do I get static libraries (.a)?</summary>

```sh
docker build \
  --build-arg V8_VERSION=9.2.24 \
  --build-arg V8_STATIC_LIB=YES \
  --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
</details>

<details>
<summary>How do I get monolithic static library (.a)? </summary>

```sh
docker build \
  --build-arg V8_VERSION=9.2.24 \
  --build-arg V8_STATIC_LIB=YES \
  --build-arg V8_MONOLITHIC=YES \
  --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
</details>
