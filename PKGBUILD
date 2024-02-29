# Maintainer: Echo J. <aidas957 at gmail dot com>
# shellcheck shell=bash disable=SC2034,SC2164

pkgname=lib32-vulkan-nouveau-git
pkgdesc="Nouveau Vulkan (NVK) EXPERIMENTAL Mesa driver with some additions (32-bit Git version)"
pkgver=24.1.0_devel.185534.4c73e529332.d41d8cd
pkgrel=1
arch=('x86_64')
depends=('lib32-libdrm' 'lib32-libxshmfence' 'lib32-libx11' 'lib32-systemd' 'lib32-vulkan-icd-loader' 'lib32-wayland')
makedepends=('elfutils' 'git' 'glslang' 'lib32-libunwind' 'lib32-libxrandr' 'lib32-rust-libs' 'meson>=1.3.0rc2'
             'python-mako' 'rust-bindgen' 'systemd' 'wayland-protocols' 'xorgproto' 'zstd')  # -rc1 has weird crate issues
optdepends=('lib32-vulkan-mesa-layers: Additional Vulkan layers'
            'linux>=6.6.arch1: Minimum required kernel for new uAPI support')
provides=('lib32-vulkan-driver')
url="https://gitlab.freedesktop.org/mesa/mesa"
license=('MIT AND BSD-3-Clause AND SGI-B-2.0')
source=("mesa::git+https://gitlab.freedesktop.org/mesa/mesa.git#commit=4c73e529332d217de79f16659d24ea2dbfd1b3ab"
        LICENSE)
sha512sums=('SKIP'
            'f9f0d0ccf166fe6cb684478b6f1e1ab1f2850431c06aa041738563eb1808a004e52cdec823c103c9e180f03ffc083e95974d291353f0220fe52ae6d4897fecc7')
install="${pkgname}.install"

pkgver() {
    cd mesa
    local _ver
    _ver=$(<VERSION)

    local _patchver
    local _patchfile
    for _patchfile in "${source[@]}"; do
        _patchfile="${_patchfile%%::*}"
        _patchfile="${_patchfile##*/}"
        [[ $_patchfile = *.patch ]] || continue
        _patchver="${_patchver}$(md5sum ${srcdir}/${_patchfile} | cut -c1-32)"
    done
    _patchver="$(echo -n $_patchver | md5sum | cut -c1-7)"

    echo ${_ver/-/_}.$(git rev-list --count HEAD).$(git rev-parse --short HEAD).${_patchver}
}

prepare() {
  # HACK: Don't copy Mesa defaults (they're basically useless for this standalone driver)
  # TODO: replace this with a build option if possible
  cd mesa
  sed -i 's/install_data/#install_data/' src/util/meson.build

  # HACK: Disable xcb-util-keysyms dependency
  # It's only used for a RADV-specific trace feature so it's useless for us
  sed -i 's/with_xcb_keysyms = dep_xcb_keysyms.found()/with_xcb_keysyms = false/' meson.build

  ### Misc stuff ###

  # Mark this NVK package with a signature (so I could track who's using it for bug report purposes)
  sed -i 's/"Mesa " PACKAGE_VERSION/"Mesa DodoNVK " PACKAGE_VERSION/' src/nouveau/vulkan/nvk_physical_device.c
}

build() {
  # Auto-download Rust crates for NAK (removes extra code for crate handling)
  _nak_crate="--force-fallback-for=syn"

  # HACK: Remove crate .rlib files before build
  # (This prevents build errors after a Rust update: https://github.com/mesonbuild/meson/issues/10706)
  [ -d build/subprojects ] && find build/subprojects -iname "*.rlib" -delete
  [ -d build/src/nouveau/compiler ] && find build/src/nouveau/compiler -iname "*.rlib" -delete

  # As you can see, I optimized the build options pretty well 🐸
  arch-meson mesa build \
    --cross-file lib32 \
    --reconfigure \
    --wrap-mode=nofallback \
    ${_nak_crate} \
    -D b_ndebug=false \
    -D platforms=x11,wayland \
    -D gallium-drivers= \
    -D vulkan-drivers=nouveau \
    -D vulkan-layers= \
    -D dri3=enabled \
    -D egl=disabled \
    -D gallium-extra-hud=false \
    -D gallium-nine=false \
    -D gallium-omx=disabled \
    -D gallium-opencl=disabled \
    -D gallium-va=disabled \
    -D gallium-vdpau=disabled \
    -D gallium-xa=disabled \
    -D gbm=disabled \
    -D gles1=disabled \
    -D gles2=disabled \
    -D glvnd=false \
    -D glx=disabled \
    -D libunwind=enabled \
    -D llvm=disabled \
    -D lmsensors=disabled \
    -D osmesa=false \
    -D shared-glapi=disabled \
    -D microsoft-clc=disabled \
    -D valgrind=disabled \
    -D android-libbacktrace=disabled \
    -D intel-rt=disabled

  meson compile -C build
}

package() {
  meson install -C build --destdir "${pkgdir}"

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}
