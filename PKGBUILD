# $Id: PKGBUILD 248014 2015-10-01 16:04:08Z fyan $
# Maintainer: Angel Velasquez <angvp@archlinux.org>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Stéphane Gaudreault <stephane@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Jason Chu <jason@archlinux.org>

pkgname=python
pkgver=3.5.0
pkgrel=2
_pybasever=3.5
pkgdesc="Next generation of the python high-level scripting language"
arch=('i686' 'x86_64')
license=('custom')
url="http://www.python.org/"
depends=('expat' 'bzip2' 'gdbm' 'openssl' 'libffi' 'zlib')
makedepends=('tk' 'sqlite' 'valgrind' 'bluez-libs' 'mpdecimal')
checkdepends=('gdb' 'xorg-server-xvfb')
optdepends=('python-setuptools'
            'python-pip'
            'sqlite'
            'mpdecimal: for decimal'
            'xz: for lzma'
            'tk: for tkinter')
options=('!makeflags')
provides=('python3')
replaces=('python3')
source=("http://www.python.org/ftp/python/${pkgver%rc*}/Python-${pkgver}.tar.xz"
        test_gdb-version-fix.patch
        dont-make-libpython-readonly.patch
        issue25150.patch)
sha1sums=('871a06df9ab70984b7398ac53047fe125c757a70'
          'ab86515aff465385675e2e6e593f09596e0a8db0'
          'c22b24324b8e53326702de439c401d97927ee3f2'
          'bd068695d22931320069200f240c425096bb5011')

prepare() {
  cd Python-${pkgver}

  # https://bugs.python.org/issue25096
  patch -p1 -i ../test_gdb-version-fix.patch

  # FS#45809
  patch -p1 -i ../dont-make-libpython-readonly.patch

  # https://bugs.python.org/issue25150
  patch -p1 -i ../issue25150.patch

  # FS#23997
  sed -i -e "s|^#.* /usr/local/bin/python|#!/usr/bin/python|" Lib/cgi.py

  # Ensure that we are using the system copy of various libraries (expat, zlib, libffi, and libmpdec),
  # rather than copies shipped in the tarball
  rm -r Modules/expat
  rm -r Modules/zlib
  rm -r Modules/_ctypes/{darwin,libffi}*
  rm -r Modules/_decimal/libmpdec
}

build() {
  cd Python-${pkgver}

  # Disable bundled pip & setuptools
  ./configure --prefix=/usr \
              --enable-shared \
              --with-threads \
              --with-computed-gotos \
              --enable-ipv6 \
              --with-system-expat \
              --with-dbmliborder=gdbm:ndbm \
              --with-system-ffi \
              --with-system-libmpdec \
              --enable-loadable-sqlite-extensions \
              --without-ensurepip

  make EXTRA_CFLAGS="$CFLAGS"
}

check() {
  # Failures:
  # test_pathlib & test_posixpath: https://bugs.python.org/issue24950

  # Hacks:
  # test_tk: xvfb-run
  # test_unicode_file: LC_CTYPE=en_US.utf-8

  cd Python-${pkgver}
  LD_LIBRARY_PATH="${srcdir}/Python-${pkgver}":${LD_LIBRARY_PATH} \
  LC_CTYPE=en_US.utf-8 xvfb-run \
    "${srcdir}/Python-${pkgver}/python" -m test.regrtest -uall || warning "Expected failure"
}

package() {
  cd Python-${pkgver}
  make DESTDIR="${pkgdir}" EXTRA_CFLAGS="$CFLAGS" install maninstall

  # Why are these not done by default...
  ln -s python3               "${pkgdir}"/usr/bin/python
  ln -s python3-config        "${pkgdir}"/usr/bin/python-config
  ln -s idle3                 "${pkgdir}"/usr/bin/idle
  ln -s pydoc3                "${pkgdir}"/usr/bin/pydoc
  ln -s python${_pybasever}.1 "${pkgdir}"/usr/share/man/man1/python.1

  # Fix FS#22552
  ln -sf ../../libpython${_pybasever}m.so \
    "${pkgdir}/usr/lib/python${_pybasever}/config-${_pybasever}m/libpython${_pybasever}m.so"

  # some useful "stuff" FS#46146
  install -dm755 "${pkgdir}"/usr/lib/python${_pybasever}/Tools/{i18n,scripts}
  install -m755 Tools/i18n/{msgfmt,pygettext}.py "${pkgdir}"/usr/lib/python${_pybasever}/Tools/i18n/
  install -m755 Tools/scripts/{README,*py} "${pkgdir}"/usr/lib/python${_pybasever}/Tools/scripts/

  # Clean-up reference to build directory
  sed -i "s|$srcdir/Python-${pkgver}:||" "$pkgdir/usr/lib/python${_pybasever}/config-${_pybasever}m/Makefile"

  # License
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
