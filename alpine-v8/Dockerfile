FROM alpine:3.13

LABEL repository.hub="alexmasterov/alpine-v8" \
      repository.url="https://github.com/AlexMasterov/dockerfiles" \
      maintainer="Alex Masterov <alex.masterow@gmail.com>"

# >= 6.1.211 | 6.1 | dev | beta | stable | latest
ARG V8_VERSION=9.2.131

ARG V8_SHELL=NO
ARG V8_MONOLITHIC=NO
ARG V8_STATIC_LIB=NO
ARG V8_SHARED_LIB=YES

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

ARG NUM_JOBS
ARG SYMBOL_LEVEL=0

ARG USE_CLANG=YES
ARG USE_LLD=YES
ARG USE_LLD_THIN_LTO=NO
ARG USE_LLD_THIN_LTO_OPT=NO
ARG LLD_THIN_LTO_JOBS

ARG USE_GN_SOURCE=NO
ARG GN_SOURCE_REV
ARG GN_EXTRA_ARGS

ARG OUT_LIBCXX=NO
ARG OUT_HEADERS=YES
ARG OUT_STRIP=YES

ENV V8_VERSION=${V8_VERSION} \
    V8_PATH=/usr/local/v8

RUN set -eux; \
  as_ver() { set -- ${1//./ }; local y="00${2:-0}" z="00${3:-0}" w="00${4:-0}"; echo "${1}${y:(-3)}${z:(-3)}${w:(-3)}"; }; \
  compare_ver() { test $(as_ver ${1}) -${2} $(as_ver ${3}); }; \
  contains() { if [ -z "${1##*${2}*}" ]; then shift 2; echo ${*:-}; fi; }; \
  get_deps() { find ${*:-} -type f -and \( -executable -or -name '*.so*' \) -exec ldd '{}' ';' | sed -rn "s/\s*(.* => )?(.*) \(.*/\2/p" | sort -u; }; \
  if_no() { if [ ${1} = 'NO' ]; then shift 1; echo ${*:-}; fi; }; \
  if_v8_ver() { if compare_ver ${V8_VERSION} ${1} ${2}; then shift 2; echo ${*:-}; fi; }; \
  if_yes() { if [ ${1} = 'YES' ]; then shift 1; echo ${*:-}; fi; }; \
  load_archive() { [ -d ${2} ] || mkdir -p ${2}; wget ${1} -T 30 -O - | tar xmz -C ${2}; }; \
  to_bool() { if [ ${1} = 'YES' ]; then echo true; else echo false; fi; }; \
  \
  case $(uname -m) in \
    x86_64)      ARCH="x64"   ;; \
    x86)         ARCH="x86"   ;; \
    armhf|armv7) ARCH="arm"   ;; \
    aarch64)     ARCH="arm64" ;; \
  esac; \
  \
  NUM_JOBS=${NUM_JOBS:-$(getconf _NPROCESSORS_ONLN)}; \
  SOURCE_REPO="https://chromium.googlesource.com"; \
  V8_SOURCE=/tmp/v8; \
  V8_DEPS=${V8_SOURCE}/DEPS; \
  \
  if [ "${V8_VERSION//[^.]}" = '.' ]; then \
    v8_version_url="https://raw.githubusercontent.com/v8/v8/${V8_VERSION}-lkgr/include/v8-version.h"; \
    v8_build_ver=$(wget "${v8_version_url}" -T 30 -O - | sed -rn "/#.*V8_BUILD_NUMBER/N;s/#.*_NUMBER (\d+).*_LEVEL (\d+)/\1.\2/p"); \
    V8_VERSION=${v8_build_ver:+${V8_VERSION}.${v8_build_ver//.0}}; \
    V8_VERSION=${V8_VERSION:-latest}; \
  fi; \
  if [ $(contains "dev beta stable latest" ${V8_VERSION} 1) ]; then \
    if [ ${V8_VERSION} = 'latest' ]; then \
      v8_version_url="https://storage.googleapis.com/chromium-v8/official/canary/v8-linux64-rel-latest.json"; \
      v8_version_regexp=".*\"version\":\s*?\"(.*\d+)\".*"; \
    else \
      v8_version_url="https://omahaproxy.appspot.com/all"; \
      v8_version_regexp="linux,${V8_VERSION}.*,(.*)$"; \
    fi; \
    \
    V8_VERSION=$(wget ${v8_version_url} -T 30 -O - | sed -rn "s/${v8_version_regexp}/\1/p"); \
  fi; \
  \
  echo -e "\
https://dl-cdn.alpinelinux.org/alpine/edge/main\n\
https://dl-cdn.alpinelinux.org/alpine/edge/community\n\
https://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories; \
  apk add --no-cache --virtual .v8-build \
    g++ \
    libexecinfo-dev \
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
  : "-- V8: download source"; \
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
    gn_rev=${GN_SOURCE_REV:-$(cat ${V8_SOURCE}/buildtools/linux64/gn.sha1 || true)}; \
    gn_rev=${gn_rev:-$(sed -rn "/['\"]gn_version['\"]:\s*/{N;s/.*['\"]git_revision:(.*)['\"],?\s*?/\1/p}" ${V8_DEPS})}; \
    gn_rev=${gn_rev:?GN_SOURCE_REV is required}; \
    : "-- GN SOURCE: download source"; \
    load_archive "https://gn.googlesource.com/gn/+archive/${gn_rev}.tar.gz" ${GN_SOURCE}; \
    : "-- GN SOURCE: stop regen config and speed up build"; \
    sed -ri \
      -e "s/(generator\s*=).*',/\1 0'/" \
      -e "/executables\['gn_unittests'\]/d" \
      -e "s/[^def] WriteGenericNinja/  del executables['gn_unittests']\n\n&/" \
      ${GN_SOURCE}/build/gen.py; \
    : "-- GN SOURCE: build"; \
    python2 ${GN_SOURCE}/build/gen.py \
      --host=linux \
      --platform=linux \
      --no-last-commit-position \
      $(if_v8_ver ge 7.9.35 --no-static-libstdc++); \
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
    ln -fs ${GN_SOURCE}/out/gn /usr/bin/gn; \
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
    sed -ri "s/\"(--target)=arm-.*\"/\"\1=$(g++ -dumpmachine)\"/" build/config/compiler/BUILD.gn; \
  fi; \
  if [ ${ARCH} = 'arm64' ]; then \
    dumpmachine=$(g++ -dumpmachine); \
    sed -ri "s/\"(--target)=aarch64-.*\"/\"\1=${dumpmachine}\"/" build/config/compiler/BUILD.gn; \
    sed -ri "s/(toolprefix) = \"aarch64-.*\"/\1 = \"\/usr\/${dumpmachine}\/bin\/\"/" build/toolchain/linux/BUILD.gn; \
  fi; \
  \
  : "-- Patch: compiler"; \
  if [ ${USE_CLANG} = 'NO' ]; then \
    if contains ${ARCH} 'arm' && compare_ver ${V8_VERSION} ge 8.0.365; then \
      sed -ri "s/if \(arm_use_neon/& \&\& is_clang/" third_party/zlib/BUILD.gn; \
    fi; \
    : "-- Patch: V8 is not -fsanitize=null clean (issue 3782)"; \
    sed -ri "s/(cflags)\s*=\s*\[(.*)\]/\1 = [\"-fno-delete-null-pointer-checks\", \2]/" BUILD.gn; \
    : "-- Patch: don't use the attribute [[nodiscard]]"; \
    sed -ri "s/(define V8_HAS_CPP_ATTRIBUTE_NODISCARD).*/\1 0/g" include/v8config.h; \
  fi; \
  if [ ${USE_CLANG} = 'YES' ] && compare_ver ${V8_VERSION} le 6.6.164; then \
    sed -ri "/(update_args|(\":|\\$|\s+)clang_revision)/{/.*/d}" build/config/compiler/BUILD.gn; \
  fi; \
  if [ "${USE_LLD}${USE_LLD_THIN_LTO}" = 'YESYES' ]; then \
    sed -ri \
      -e "/(cache_policy =.*\"|.*(lld|thin)lto-?cache.*,?)/d" \
      -e "s/(thinlto-jobs=).*(\",?)/\1${USE_LLD_THIN_LTO_JOBS:-${NUM_JOBS}}\2/" \
      build/config/compiler/BUILD.gn; \
  fi; \
  \
  : "-- Patch: V8"; \
  if contains ${ARCH} 'arm'; then \
    sed -ri "s/(V8_HOST_ARCH)_PPC \|\| V8_HOST_ARCH_PPC64/& \|\| \1_ARM \|\| \1_ARM64/" src/base/cpu.cc; \
  fi; \
  if compare_ver ${V8_VERSION} le 7.4.221; then \
    : "-- Patch: fix deadlock in OptimizingCompileDispatcher (issue 935358)"; \
    sed -ri "/.*access_output_queue_.*;\s*/{N;s/(.*output_queue_\.push.*)/{\n\1\n}/}" \
      src/compiler-dispatcher/optimizing-compile-dispatcher.cc; \
  fi; \
  : "-- Patch: use monotonic clock for 'pthread_cond_timedwait' with musl"; \
  sed -i "s/(V8_OS_LINUX && V8_LIBC_GLIBC)/V8_OS_LINUX/g" src/base/platform/condition-variable.cc; \
  : "-- Patch: fix undefined symbol: backtrace_symbols (execinfo.h)"; \
  sed -ri "/config\(\"libbase_config\"\)/{;:a;N;/libs/!ba;s/(libs) = \[(.*)\]/\1 = \[\"execinfo\", \2\]/}" BUILD.gn; \
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
      v8_target_cpu=\"${ARCH}\" \
      is_debug=$(to_bool ${V8_DEBUG}) \
      $(if_yes ${V8_DEBUG} v8_optimized_debug=$(to_bool ${V8_DEBUG_OPT})) \
      $(if_no  ${V8_DEBUG} v8_enable_disassembler=$(to_bool ${V8_DISASSEMBLER})) \
      v8_symbol_level=${V8_SYMBOL_LEVEL:-1} \
      \
      v8_static_library=$(to_bool ${V8_STATIC_LIB}) \
      $(if_yes ${V8_MONOLITHIC} \
        is_component_build=false \
        v8_monolithic=true \
        v8_use_external_startup_data=false \
      ) \
      $(if_no ${V8_MONOLITHIC} \
        is_component_build=$(to_bool ${V8_SHARED_LIB}) \
        v8_use_external_startup_data=$(to_bool ${V8_EXTERNAL_STARTUP_DATA}) \
      ) \
      \
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
    $(if_yes ${V8_MONOLITHIC} v8_monolith) \
    $(if_no  ${V8_MONOLITHIC} \
      $(if_yes ${V8_SHARED_LIB} v8) \
    ); \
  \
  mkdir -p ${V8_PATH}; \
  if [ ${OUT_HEADERS} = 'YES' ]; then \
    : "-- Out: V8 headers"; \
    find ${V8_SOURCE}/include/ -depth \( -type f -not -name '*.h' -or -type d -empty \) -delete; \
    chmod -R 644 ${V8_SOURCE}/include/; \
    mv include ${V8_PATH}; \
  fi; \
  if [[ ${V8_STATIC_LIB} = 'YES' || ${V8_MONOLITHIC} = 'YES' ]]; then \
    : "-- Out: V8 static libraries"; \
    find ${V8_SOURCE}/out/obj/ -depth \( -type f -not -name '*.a' -or -type d -empty \) -delete; \
    chmod -R 755 ${V8_SOURCE}/out/obj/; \
    mv ${V8_SOURCE}/out/obj ${V8_PATH}; \
  fi; \
  if [[ ${V8_SHARED_LIB} = 'YES' && ${V8_MONOLITHIC} = 'NO' ]]; then \
    : "-- Out: V8 shared libraries"; \
    mkdir -p ${V8_PATH}/lib; \
    find ${V8_SOURCE}/out/ -maxdepth 1 -type f -regex '.*.\(so\|bin\|dat\)' \
      -exec mv '{}' ${V8_PATH}/lib +; \
    \
    if [ ${OUT_LIBCXX} = 'YES' ]; then \
      : "-- Out: C++ shared libraries"; \
      out_libs=$(get_deps ${V8_PATH}/lib | sed -rn "s/^\/usr\/lib\//&/p"); \
      cp --dereference ${out_libs} ${V8_PATH}/lib; \
    fi; \
    \
    chmod -R 755 ${V8_PATH}/lib/; \
    ln -fs ${V8_PATH}/lib/*.so* /usr/local/lib; \
  fi; \
  if [ ${V8_SHELL} = 'YES' ]; then \
    : "-- Out: V8 executables"; \
    mkdir -p ${V8_PATH}/bin; \
    mv ${V8_SOURCE}/out/d8 ${V8_PATH}/bin/d8; \
    ln -fs ${V8_PATH}/bin/d8 /usr/local/bin/d8; \
    \
    if [[ ${V8_SHARED_LIB} = 'NO' || "${V8_SHARED_LIB}${OUT_LIBCXX}" = 'YESNO' ]]; then \
      d8_std_libs=$(get_deps ${V8_PATH}/bin/d8 | sed -rn "s/\/usr\/lib\/(.*)/so:\1/p"); \
      apk add --no-cache --virtual .d8-runtime ${d8_std_libs}; \
    fi; \
  fi; \
  if [ ${OUT_STRIP} = 'YES' ]; then \
    : "-- Out: strip executables"; \
    find ${V8_PATH} -type f -executable -not -name '*.a' \
      -exec strip --strip-unneeded '{}' + || true; \
  fi; \
  \
  : "-- Remove build dependencies, clean temporary files"; \
  apk del --no-network .v8-build; \
  rm -rf /tmp/* /var/tmp/*

WORKDIR ${V8_PATH}
