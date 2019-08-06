# Maintainer: Filipe Laíns (FFY-01) <lains@archlinux.org>
# Maintainer: Daniel Kozak (kozzi) <kozzi11@gmail.com>
# Contributor: Mihails Strasuns <public@dicebot.lv>
# Contributor: Moritz Maxeiner <moritz@ucworks.org>
# Contributor: Jerome Berger <jeberger@free.fr>
# Contributor: Jesus Alvarez <jeezusjr@gmail.com>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Elijah Stone <elronnd@elronnd.net>

pkgbase=gdc-git
pkgname=(gdc-git libgphobos-git)
pkgver=10.0.0+2.086.0
_branch=ibuclaw/gdc # Change here! pkgver/_gccver/_d_ver will be automatically updated.
_islver=0.21 # Change here!
_d_ver=''
pkgrel=3
arch=('x86_64' 'i686')
license=('GPL3')
url="https://github.com/D-Programming-GDC/GDC"
pkgdesc="GCC based D compiler"
groups=('dlang')
makedepends=('git' 'gdc')
source=(
        "http://isl.gforge.inria.fr/isl-$_islver.tar.bz2"
        "gdc::git+https://github.com/gcc-mirror/gcc.git#branch=$_branch"
        'git+https://github.com/D-Programming-GDC/GDMD.git'
        'paths.diff')
sha256sums=('777058852a3db9500954361e294881214f6ecd4b594c00da5eee974cd6a54960'
            'SKIP'
            'SKIP'
            '841504e9dffe718f7e5a5fbbf03299f2b51acd783d47f99894aa5d411abcc56a')

pkgver() {
  if [ -f gdc/gcc/d/dmd/VERSION ]; then
    _d_ver="+$(cat gdc/gcc/d/dmd/VERSION | sed 's|\"||g')"
  fi
  echo "$(cat gdc/gcc/BASE-VER)$_d_ver"
}

prepare() {
  # Setup paths
  ln -sf "$srcdir"/isl-$_islver "$srcdir"/gcc/isl

  # Setup gcc
  cd "$srcdir"/gdc

  ln -s ../isl-${_islver} isl

  # Do not run fixincludes
  sed -i 's@\./fixinc\.sh@-c true@' gcc/Makefile.in

  # Arch Linux installs x86_64 libraries /lib
  sed -i '/m64=/s/lib64/lib/' gcc/config/i386/t-linux64

  # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

  git apply "$srcdir"/paths.diff

  mkdir -p "$srcdir/gcc-build"

}

build() {
  cd "$srcdir"/gcc-build

  # using -pipe causes spurious test-suite failures
  # http://gcc.gnu.org/bugzilla/show_bug.cgi?id=48565
  export CFLAGS="${CFLAGS/-pipe/} -O2"
  export CXXFLAGS="${CXXFLAGS/-pipe/} -O2"

  "$srcdir"/gcc/configure --prefix=/usr \
                          --libdir=/usr/lib \
                          --libexecdir=/usr/lib \
                          --mandir=/usr/share/man \
                          --infodir=/usr/share/info \
                          --enable-languages=d \
                          --enable-shared \
                          --enable-static \
                          --enable-threads=posix \
                          --enable-libmpx \
                          --with-system-zlib \
                          --with-isl \
                          --enable-__cxa_atexit \
                          --disable-libunwind-exceptions \
                          --enable-clocale=gnu \
                          --disable-libstdcxx-pch \
                          --disable-libssp \
                          --enable-gnu-unique-object \
                          --enable-linker-build-id \
                          --enable-lto \
                          --enable-plugin \
                          --enable-install-libiberty \
                          --with-linker-hash-style=gnu \
                          --enable-gnu-indirect-function \
                          --disable-multilib \
                          --disable-werror \
                          --disable-bootstrap \
                          --enable-default-pie \
                          --enable-default-ssp \
                          --with-bugurl=https://bugzilla.gdcproject.org/ \
                          --with-pkgversion="GDC ${pkgver%+*} based on D v${pkgver#*+} built with ISL $_islver for Arch Linux" \
                          gdc_include_dir=/usr/include/dlang/gdc

  make $MAKEFLAGS
}

package_gdc-git() {
  pkgdesc="Compiler for D programming language which uses gcc backend"
  depends=('gcc' 'perl' 'binutils' 'libgphobos')
  provides=("d-compiler=${pkgver#*+}" 'gdc')
  conflicts=('gdc')

  # Binaries
  install -Dm 755 gcc-build/gcc/gdc "$pkgdir"/usr/bin/gdc
  install -Dm 755 gcc-build/gcc/cc1d "$pkgdir"/usr/lib/gcc/$CHOST/${pkgver%+*}/cc1d
  install -Dm 755 GDMD/dmd-script "$pkgdir"/usr/bin/gdmd

  # Doc
  install -Dm 644 "$srcdir"/GDMD/dmd-script.1 "$pkgdir"/usr/share/man/man1/gdmd.1
}


package_libgphobos-git() {
  pkgdesc="Standard library for D programming language, GDC port"
  provides=('d-runtime' 'd-stdlib' 'libgphobos')
  conflicts=('libgphobos')
  options=('staticlibs')

  cd "$srcdir"/gcc-build
  make -C $CHOST/libphobos DESTDIR="$pkgdir" install

  if [ -d "$pkgdir"/usr/lib64 ]; then
    mv "$pkgdir"/usr/lib64/* "$pkgdir"/usr/lib
    rmdir "$pkgdir"/usr/lib64
  fi
}
