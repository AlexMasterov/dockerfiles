# dockerfiles

Common Dockerfiles, used for _development_.

- [alpine-v8](#alpine-v8)
- [alpine-php](#alpine-php)

#### alpine-v8

```sh
docker build --build-arg V8_VERSION=stable --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
> V8_VERSION >= 6.1.211 | 6.1 | dev | beta | stable | latest

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
  --build-arg V8_VERSION=stable \
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
  --build-arg V8_VERSION=stable \
  --build-arg V8_STATIC_LIB=YES \
  --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
</details>

<details>
<summary>How do I get monolithic static library (.a)?</summary>

```sh
docker build \
  --build-arg V8_VERSION=stable \
  --build-arg V8_MONOLITHIC=YES \
  --rm -t v8 github.com/AlexMasterov/dockerfiles.git#:alpine-v8
```
</details>

<details>
<summary>I have build issues</summary>

I get this error message:
```sh
ERROR at //gni/v8.gni:103:3: Dependency not allowed.
  target(link_target_type, target_name) {
  ^--------------------------------------
The item //src/inspector:inspector
can not depend on //:features
because it is not in //:features's visibility list: [
  //.:*
]
```
Try to use:
1.  `--build-arg USE_GN_SOURCE=YES`
2.  `--build-arg USE_GN_SOURCE=YES GN_SOURCE_REV=9434c3d281eefb5c3764b5a0575feec9c59bd095`

> `GN_SOURCE_REV` can be any revision on the _working_ repository: [https://gn.googlesource.com/gn](gn.googlesource.com/gn).

</details>

#### alpine-php
```sh
docker build --build-arg PHP_VERSION=8.0 --rm -t php github.com/AlexMasterov/dockerfiles.git#:alpine-php
```
> PHP_VERSION >= 7.0.0 | 7.0
