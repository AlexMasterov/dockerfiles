FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-v8" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 6.1.211
ARG V8_VERSION=9.2.11

ARG V8_SHELL=NO
ARG V8_STATIC=NO
ARG V8_MONOLITHIC=NO

ARG V8_DEBUG=NO
ARG V8_DEBUG_OPT=NO
ARG V8_SYMBOL_LEVEL=0

ARG V8_INTL=NO
ARG V8_SIPHASH=NO
ARG V8_OBJECT_PRINT=NO
ARG V8_DISASSEMBLER=NO
ARG V8_POINTER_COMPRESSION=YES
ARG V8_EXTERNAL_STARTUP_DATA=NO
ARG V8_31BIT_SMIS_ON_64BIT_ARCH=YES
ARG V8_UNTRUSTED_CODE_MITIGATIONS=NO
ARG V8_TYPED_ARRAY_MAX_SIZE_IN_HEAP=64

ARG SYMBOL_LEVEL=0

ARG USE_CLANG=YES
ARG USE_LLD=YES
ARG USE_LLD_THIN_LTO=NO
ARG USE_LLD_THIN_LTO_OPT=NO

ARG NUM_JOBS
ARG LLD_THIN_LTO_JOBS

ARG USE_GN_SOURCE=NO
ARG GN_SOURCE_REV
ARG GN_SOURCE_REPO=https://gn.googlesource.com/gn
ARG GN_EXTRA_ARGS

ARG SOURCE_REPO=https://chromium.googlesource.com

ARG OUT_LIBCXX=NO
ARG OUT_HEADERS=YES
ARG OUT_STRIP=YES

ENV V8_VERSION=${V8_VERSION} \
    V8_PATH=/usr/local/v8

RUN set -eux; \
  V8_SOURCE=/tmp/v8; \
  V8_DEPS=${V8_SOURCE}/DEPS; \
  NUM_JOBS=${NUM_JOBS:-$(getconf _NPROCESSORS_ONLN)}; \
  case $(cat /etc/apk/arch) in \
    x86_64)      ARCH="x64" ;; \
    x86)         ARCH="x86" ;; \
    armhf|armv7) ARCH="arm" ;; \
  esac; \
  \
  load_archive() { [ -d ${2} ] || mkdir -p "${2}"; wget ${1} -T 30 -O - | tar xmz -C "${2}"; }; \
  get_deps() { scanelf --needed --nobanner -F '%n#p' "${*:-}" | tr ',' '\n' | sed -r "/($(ls -1 ${V8_PATH}/lib | paste -sd '|'))/d" | sort -u; }; \
  as_ver() { set -- ${1//./ }; local y="00${2:-0}" z="00${3:-0}" w="00${4:-0}"; echo "${1}${y:(-3)}${z:(-3)}${w:(-3)}"; }; \
  compare_ver() { test $(as_ver ${1}) -${2} $(as_ver ${3}); }; \
  if_v8_ver() { if compare_ver ${V8_VERSION} ${1} ${2}; then shift 2; echo "${*:-}"; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo "${*:-}"; fi; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo "${*:-}"; fi; }; \
  to_bool() { if [ ${1} = 'YES' ]; then echo true; else echo false; fi; }; \
  contains() { if [ -z ${1##*${2}*} ]; then shift 2; echo "${*:-}"; fi; }; \
  \
  echo -e "\
https://dl-cdn.alpinelinux.org/alpine/edge/main\n\
https://dl-cdn.alpinelinux.org/alpine/edge/community\n\
https://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories; \
  apk add --no-cache --virtual .v8-build \
    g++ \
    linux-headers \
    python2 \
    samurai \
    $(if_yes ${USE_CLANG} \
      clang=~11 \
      llvm11=~11 \
    ) \
    $(if_no  ${USE_CLANG}     gcc) \
    $(if_yes ${USE_LLD}       lld=~11) \
    $(if_yes ${USE_GN_SOURCE} clang=~11) \
    $(if_no  ${USE_GN_SOURCE} gn) \
    $(if_yes ${V8_INTL}       icu-dev) \
    ; \
  : "-- V8: download"; \
  load_archive "${SOURCE_REPO}/v8/v8/+archive/${V8_VERSION}.tar.gz" ${V8_SOURCE}; \
  : "-- V8: download deps."; \
  deps=" \
    base/trace_event/common \
    build \
    buildtools \
    $(if_yes ${V8_INTL} third_party/icu) \
    $(if_yes ${V8_SHELL} \
      third_party/jinja2 \
      third_party/markupsafe \
    ) \
    $(if_v8_ver le 6.7.148 testing/gtest) \
    $(if_v8_ver ge 6.7.149 third_party/googletest/src) \
    $(if_v8_ver ge 8.0.365 third_party/zlib) \
  "; \
  for dep in ${deps}; do { \
    target="${dep//\//\\/}"; \
    target="${target//+/\\+}"; \
    repo=$(sed -rn "/['\"].*${target}['\"]:\s*/{N;s/.*['\"](.*).git['\"].*,?/\1/p}" ${V8_DEPS}); \
    rev=$(sed -rn "/['\"].*${target}['\"]:\s*/{N;s/.*['\"](.*)['\"],?/\1/p}" ${V8_DEPS}); \
    \
    load_archive "${SOURCE_REPO}${repo}/+archive/${rev}.tar.gz" ${V8_SOURCE}/${dep}; \
  } & \
  done; \
  wait; \
  \
  if [ ${USE_GN_SOURCE} = 'YES' ]; then \
    GN_SOURCE=/tmp/gn; \
    gn_rev=${GN_SOURCE_REV:-$(sed -rn "/['\"]gn_version['\"]:\s*/{N;s/.*['\"]git_revision:(.*)['\"],?/\1/p}" ${V8_DEPS})}; \
    gn_rev=${gn_rev:?GN_SOURCE_REV is required}; \
    : "-- GN SOURCE: download"; \
    load_archive "${GN_SOURCE_REPO}/+archive/${gn_rev}.tar.gz" ${GN_SOURCE}; \
    : "-- GN SOURCE: stop regen config and speedup build"; \
    sed -ri \
      -e "s/(generator\s*=).*',/\1 0'/" \
      -e "/executables\['gn_unittests'\]/d" \
      -e "s/[^def] WriteGenericNinja/  del executables['gn_unittests']\n\n&/" \
      ${GN_SOURCE}/build/gen.py; \
    : "-- GN SOURCE: build"; \
    python2 ${GN_SOURCE}/build/gen.py \
      --host=linux \
      --platform=linux \
      --no-static-libstdc++ \
      --no-last-commit-position; \
    echo -e "\
    \n#ifndef OUT_LAST_COMMIT_POSITION_H_ \
    \n#define OUT_LAST_COMMIT_POSITION_H_ \
    \n#define LAST_COMMIT_POSITION_NUM 0 \
    \n#define LAST_COMMIT_POSITION \"${gn_rev::8}\" \
    \n#endif \
    " > ${GN_SOURCE}/out/last_commit_position.h; \
    ninja \
      -C ${GN_SOURCE}/out \
      -j ${NUM_JOBS}; \
    ln -f ${GN_SOURCE}/out/gn /usr/bin/gn; \
  fi; \
  \
  cd ${V8_SOURCE}; \
  : "-- Patch: build"; \
  sed -ri \
    -e "$(if_no ${V8_SHELL} "s/\"(src|test)\/inspector:.*\",?//")" \
    -e "s/\"(tools|test):gn_all\",?//g" \
    -e "s/\"test\/(cctest|unittests):.*\",?//g" \
    -e "s/\"\/\/build\/(win:default_exe_manifest|config:exe_and_shlib_deps)\",?//" \
    BUILD.gn; \
  if compare_ver ${V8_VERSION} le 6.7.197; then \
    sed -ri "s/\"\/\/test\/test262\/BUILD.gn\",?//" .gn; \
  fi; \
  if [ ${V8_INTL} = 'NO' ] && compare_ver ${V8_VERSION} le 6.7.196; then \
    sed -i "s/import(\"\/\/third_party\/icu\/config.gni\")/icu_use_data_file = false/" gni/isolate.gni; \
  fi; \
  if compare_ver ${V8_VERSION} le 7.1.165 && compare_ver ${V8_VERSION} ge 7.1.79; then \
    sed -i "/import(\"\/\/third_party\/catapult\/devil\/devil_arm\.gni\")/d" build/config/arm.gni; \
  fi; \
  if compare_ver ${V8_VERSION} le 7.2.505 && compare_ver ${V8_VERSION} ge 7.1.78; then \
    sed -i "s/if (use_atk)/if (false)/" build/config/linux/BUILD.gn; \
  fi; \
  if compare_ver ${V8_VERSION} ge 8.6.51; then \
    echo "checkout_google_benchmark = false" >> build/config/gclient_args.gni; \
  fi; \
  if compare_ver ${V8_VERSION} le 8.7.209; then \
    sed -ri "s/(set_sources_assignment_filter)\(.*\)/\1([])/" build/config/BUILDCONFIG.gn; \
  fi; \
  if [ ${ARCH} = 'arm' ]; then \
    sed -ri "s/(V8_HOST_ARCH)_PPC \|\| V8_HOST_ARCH_PPC64/& || \1_ARM/" src/base/cpu.cc; \
    sed -ri "s/\"(--target)=arm-linux-gnueabihf\"/\"\1=$(g++ -dumpmachine)\"/" build/config/compiler/BUILD.gn; \
  fi; \
  \
  : "-- Patch: compiler"; \
  if [ ${USE_CLANG} = 'NO' ]; then \
    : "-- Patch: V8 is not -fsanitize=null clean (issue 3782)"; \
    sed -ri "s/(cflags)\s*=\s*\[(.*)\]/\1 = [\"-fno-delete-null-pointer-checks\", \2]/" BUILD.gn; \
  fi; \
  if [ ${USE_CLANG} = 'YES' ] && compare_ver ${V8_VERSION} le 6.6.164; then \
    sed -ri "/(update_args|(\":|\\$|\s+)clang_revision)/s/.*//" build/config/compiler/BUILD.gn; \
  fi; \
  if [ "${USE_LLD}${USE_LLD_THIN_LTO}" = 'YESYES' ]; then \
    sed -ri \
      -e "s/(cache_policy =.*\"|.*(lld|thin)lto-?cache.*,?)//" \
      -e "s/(thinlto-jobs=).*(\",?)/\1${USE_LLD_THIN_LTO_JOBS:-${NUM_JOBS}}\2/" \
      build/config/compiler/BUILD.gn; \
  fi; \
  \
  : "-- Patch: V8"; \
  if compare_ver ${V8_VERSION} le 7.4.221; then \
    : "-- Patch: fix deadlock in OptimizingCompileDispatcher (issue 935358)"; \
    sed -ri "/.*access_output_queue_.*;\s*/{N;s/(.*output_queue_\.push.*)/{\n\1\n}/}" \
      src/compiler-dispatcher/optimizing-compile-dispatcher.cc; \
  fi; \
  : "-- Patch: use monotonic clock for 'pthread_cond_timedwait' with musl"; \
  sed -i "s/(V8_OS_LINUX && V8_LIBC_GLIBC)/V8_OS_LINUX/g" src/base/platform/condition-variable.cc; \
  : "-- Patch: fix undefined symbol: backtrace_symbols (execinfo.h)"; \
  sed -ri "s/^#if defined\(DEBUG\).*/#if 0/" src/$(if_v8_ver ge 7.6.234 codegen/)/external-reference-table.cc; \
  sed -ri \
    -e "s/^#if V8_LIBC_GLIBC.*/#if 0/" \
    -e "/const char k(MangledSymbolPrefix|SymbolCharacters).*/{N;s/.*\";//}" \
    src/base/debug/stack_trace_posix.cc; \
  : "-- GN: generate instructions"; \
  gn gen out \
    --script-executable=/usr/bin/python2 \
    --args=" \
      target_cpu=\"${ARCH}\" \
      target_os=\"linux\" \
      symbol_level=${SYMBOL_LEVEL:-1} \
      \
      is_cfi=false \
      is_clang=$(to_bool ${USE_CLANG}) \
      $(if_yes ${USE_CLANG} \
        clang_base_path=\"/usr\" \
        clang_use_chrome_plugins=false \
      ) \
      use_sysroot=false \
      use_custom_libcxx=false \
      use_custom_libcxx_for_host=false \
      use_glib=false \
      use_gold=false \
      use_lld=$(to_bool ${USE_LLD}) \
      $(if_yes ${USE_LLD} \
        use_thin_lto=$(to_bool ${USE_LLD_THIN_LTO}) \
        thin_lto_enable_optimizations=$(to_bool ${USE_LLD_THIN_LTO_OPT}) \
      ) \
      fatal_linker_warnings=false \
      treat_warnings_as_errors=false \
      \
      is_debug=$(to_bool ${V8_DEBUG}) \
      $(if_yes ${V8_DEBUG} v8_optimized_debug=$(to_bool ${V8_DEBUG_OPT})) \
      $(if_no  ${V8_DEBUG} v8_enable_disassembler=$(to_bool ${V8_DISASSEMBLER})) \
      $(if_yes ${V8_MONOLITHIC} \
        is_component_build=false \
        v8_monolithic=true \
        v8_use_external_startup_data=false \
      ) \
      $(if_no ${V8_MONOLITHIC} \
        is_component_build=true \
        v8_use_external_startup_data=$(to_bool ${V8_EXTERNAL_STARTUP_DATA}) \
      ) \
      v8_static_library=$(to_bool ${V8_STATIC}) \
      v8_symbol_level=${V8_SYMBOL_LEVEL:-1} \
      \
      v8_target_cpu=\"${ARCH}\" \
      v8_use_siphash=$(to_bool ${V8_SIPHASH}) \
      v8_win64_unwinding_info=false \
      v8_enable_backtrace=true \
      v8_enable_gdbjit=false \
      v8_enable_vtunejit=false \
      v8_enable_vtunetracemark=false \
      v8_enable_handle_zapping=false \
      v8_enable_google_benchmark=false \
      v8_enable_i18n_support=$(to_bool ${V8_INTL}) \
      v8_enable_object_print=$(to_bool ${V8_OBJECT_PRINT}) \
      v8_typed_array_max_size_in_heap=${V8_TYPED_ARRAY_MAX_SIZE_IN_HEAP:-64} \
      v8_untrusted_code_mitigations=$(to_bool ${V8_UNTRUSTED_CODE_MITIGATIONS}) \
      $(contains ${ARCH} '64' \
        v8_enable_pointer_compression=$(to_bool ${V8_POINTER_COMPRESSION}) \
        v8_enable_31bit_smis_on_64bit_arch=$(to_bool ${V8_31BIT_SMIS_ON_64BIT_ARCH}) \
      ) \
      ${GN_EXTRA_ARGS:-} \
    "; \
  : "-- Ninja: build V8"; \
  ninja \
    -C ${V8_SOURCE}/out \
    -j ${NUM_JOBS} \
    $(if_yes ${V8_SHELL}      d8) \
    $(if_no  ${V8_MONOLITHIC} v8) \
    $(if_yes ${V8_MONOLITHIC} v8_monolith); \
  \
  mkdir -p ${V8_PATH}; \
  if [ ${OUT_HEADERS} = 'YES' ]; then \
    : "-- Out: V8 headers"; \
    find ${V8_SOURCE}/include/ -depth \( -type f -not -name "*.h" -or -type d -empty \) -delete; \
    chmod -R 644 ${V8_SOURCE}/include/; \
    mv include ${V8_PATH}; \
  fi; \
  if [[ ${V8_MONOLITHIC} = 'YES' || ${V8_STATIC} = 'YES' ]]; then \
    : "-- Out: V8 static libraries"; \
    find ${V8_SOURCE}/out/obj/ -depth \( -type f -not -name "*.a" -or -type d -empty \) -delete; \
    chmod -R 755 ${V8_SOURCE}/out/obj/; \
    mv ${V8_SOURCE}/out/obj ${V8_PATH}; \
  fi; \
  if [ ${V8_MONOLITHIC} = 'NO' ]; then \
    : "-- Out: V8 shared libraries"; \
    mkdir -p ${V8_PATH}/lib; \
    find ${V8_SOURCE}/out/ -type f -regex ".*.\(so\|bin\|dat\)" -maxdepth 1 \
      -exec mv '{}' ${V8_PATH}/lib +; \
    \
    if [ ${OUT_LIBCXX} = 'YES' ]; then \
      : "-- Out: C++ shared libraries"; \
      v8_std_libs=$(get_deps ${V8_PATH}/lib | paste -sd '|'); \
      std_libs=$(find /usr/lib -regex ".*\(${v8_std_libs//\|/\\\|}\)" -and \( -type f -or -type l \)); \
      \
      out_libs=""; \
      for std_lib in ${std_libs}; do \
        std_lib_deps=$(ldd ${std_lib} | sed -rn "s/.*=> (\/usr\/lib\/.*) \(.*\)/\1/p"); \
        out_libs="${std_lib} ${std_lib_deps} ${out_libs}"; \
      done; \
      out_libs=$(echo ${out_libs} | tr ' ' '\n' | sort -u); \
      \
      cp --dereference ${out_libs} ${V8_PATH}/lib; \
    fi; \
    \
    chmod -R 755 ${V8_PATH}/lib/; \
    ln -f ${V8_PATH}/lib/*.so* /usr/local/lib; \
  fi; \
  if [ ${V8_SHELL} = 'YES' ]; then \
    : "-- Out: V8 executables"; \
    mkdir -p ${V8_PATH}/bin; \
    mv ${V8_SOURCE}/out/d8 ${V8_PATH}/bin/d8; \
    ln -f ${V8_PATH}/bin/d8 /usr/local/bin/d8; \
    \
    if [[ ${V8_MONOLITHIC} = 'YES' || "${V8_MONOLITHIC}${OUT_LIBCXX}" = 'NONO' ]]; then \
      d8_std_libs=$(get_deps ${V8_PATH}/bin/d8 | sed "s/^/so:&/"); \
      apk add --no-cache --virtual .d8-runtime ${d8_std_libs}; \
    fi; \
  fi; \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    find ${V8_PATH} -type f -executable -not -name "*.a" \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .v8-build; \
  rm -rf /tmp/* /var/tmp/*

WORKDIR ${V8_PATH}