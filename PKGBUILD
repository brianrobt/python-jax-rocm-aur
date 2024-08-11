# Based on https://github.com/google/jax/blob/main/build/rocm/README.md
# Based on python-jax, python-jax-opt-cuda-git
#
# Maintainer: Brian Thompson <brianrobt@pm.me>
#
# Previous contributors:
# Contributor: wuxxin <wuxxin@gmail.com>
#
# Original contributors:
# Contributor: Daniel Bershatsky <bepshatsky@yandex.ru>

pkgname='python-jax-rocm'
pkgver=0.4.31
pkgrel=1
pkgdesc='XLA library for JAX (jaxlib for ROCM)'
_srcname="jax-jaxlib-v${pkgver}"
_xlaname="xla-rocm-jaxlib-v${pkgver}"
arch=('x86_64')
url='https://github.com/google/jax'
license=('Apache')
groups=('jax')
depends=(
  'python-absl'
  'python-flatbuffers'
  'python-ml-dtypes'
  'python-numpy'
  'python-scipy'
  'miopen'
  'rccl'
  'rocm-hip-runtime'
)
makedepends=(
  'python-build'
  'python-installer'
  'python-setuptools'
  'python-wheel'
  'bazel'
  'miopen'
  'rccl'
  'rocm-hip-sdk'
)
conflicts=('python-jaxlib')
provides=("python-jaxlib=$pkgver")
source=(
  "${_srcname}.tar.gz::https://github.com/google/jax/archive/refs/tags/jaxlib-v${pkgver}.tar.gz"
  "${_xlaname}.tar.gz::https://github.com/ROCmSoftwarePlatform/xla/archive/refs/heads/rocm-jaxlib-v${pkgver}.tar.gz"
)
sha256sums=(
  '022ea1347f9b21cbea31410b3d650d976ea4452a48ea7317a5f91c238031bf94'
  '2ea3a38396a679cb6bd4e737c759f87357ba4bc20c776a1c61e90f09a4c52779'
)

# test
# python -c "import jax; print(jax.devices(),jax.devices()[0].device_kind); x=jax.numpy.array([1.2,3.4,5.6]); y=jax.numpy.exp(x); print(y)"

prepare() {
  cd "${srcdir}/${_srcname}"
  # loosen acceptable bazel version
  echo "6.*.*" >.bazelversion
  # Override default version
  export JAXLIB_RELEASE=$pkgver
  # export ROCM_HOME if not set
  if test -n "$ROCM_HOME"; then
    export ROCM_HOME=/opt/rocm
  fi
}

build() {
  cd "${srcdir}/${_srcname}"
  # populate build architecture list if not set from aur:tensorflow-rocm@2.13.0-4
  if test -n "$TF_ROCM_AMDGPU_TARGETS"; then
    export TF_ROCM_AMDGPU_TARGETS="$TF_ROCM_AMDGPU_TARGETS"
  else
    export TF_ROCM_AMDGPU_TARGETS="gfx803,gfx900,gfx906,gfx908,gfx90a,gfx1030,gfx1100,gfx1101,gfx1102"
  fi

  # XXX use xla tree from ROCmSoftwarePlatform:xla@v$pkgver with fixes for rocm
  # FIXME hipcc and hipcc.perl is searched in $ROCM_HOME/hip/bin but only available in $ROCM_HOME/bin
  # FIXME include/hipblaslt/hipblaslt.h not found
  python build/build.py --enable_rocm \
    "--rocm_amdgpu_targets=${TF_ROCM_AMDGPU_TARGETS}" \
    --bazel_options=--override_repository=xla=${srcdir}/${_xlaname}
}

package() {
  cd "${srcdir}/${_srcname}"
  python -m installer \
    --compile-bytecode 1 \
    --destdir $pkgdir \
    $srcdir/$_srcname/dist/jaxlib-$pkgver-*.whl
}
