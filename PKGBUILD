# Maintainer: Tattwamashi Nayak <tattwamashi386@gmail.com>
#
# libfprint with an open-source driver for the FocalTech FT9366 fingerprint
# sensor behind a Realtek RTS5811 USB bridge (USB 2808:a658, ASUS VivoBook and
# others). Builds the official libfprint release with every upstream driver,
# plus the ft9366 driver from ft9366-driver.patch. No proprietary code.
#
# Driver source and issues: https://github.com/tattwamashi/libfprint (branch
# ft9366); the patch is `git diff 6f9479c ft9366` there. Based on the ft9366
# driver by V8V88V8V88. Build recipe follows Arch's libfprint package.

pkgname=libfprint-ft9366
_pkgname=libfprint
pkgver=1.94.100
pkgrel=1
pkgdesc="Library for fingerprint readers, with an open driver for FocalTech FT9366 (2808:a658)"
url="https://fprint.freedesktop.org/"
arch=(x86_64)
license=(LGPL-2.1-or-later)
depends=(
  libgcc
  glib2
  glibc
  libgudev
  libgusb
  openssl
  pixman
)
makedepends=(
  git
  glib2-devel
  gobject-introspection
  gtk-doc
  meson
  python-cairo
  python-gobject
  systemd
)
checkdepends=(
  cairo
  umockdev
)
provides=("libfprint=$pkgver" libfprint-2.so)
conflicts=(libfprint)
groups=(fprint)
source=("git+https://gitlab.freedesktop.org/libfprint/libfprint.git#tag=v$pkgver"
        ft9366-driver.patch)
b2sums=('56cbe62a6d4b98d1de90f45bae8de605418802d1b4d8d5dc55a8557a65045b115df51daf8385decdebd8aea79c86103467d5a327cbd461030d450f0e15466d08'
        'da43fc1e8d65757cf2d35424318180d2b9bc578d913ed726900ee02b94462074daf3378334d4093773ff0107636e7952ff788db214d580cc7807da309d6e572f')

prepare() {
  cd $_pkgname
  git apply "$srcdir/ft9366-driver.patch"
}

build() {
  local meson_options=(
    # Add virtual drivers for integration tests (e.g. in fprintd)
    -D drivers=all

    -D installed-tests=false
  )

  arch-meson $_pkgname build "${meson_options[@]}"
  meson compile -C build
}

check() {
  # Replayed USB recordings (umockdev) stall when every core is busy, which
  # makes e.g. the egis_etu905 test fail at random on many-core machines.
  meson test -C build --print-errorlogs --num-processes 4
}

package() {
  meson install -C build --destdir "$pkgdir"
}

# vim:set sw=2 sts=-1 et:
