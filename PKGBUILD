# Maintainer:  Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Daniel Kozak <kozzi11@gmail.com>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: libtool requires rebuilt with each new gcc version

pkgname=(gcc gcc-libs gcc-d)
pkgver=12.0.0
_majorver=${pkgver%%.*}
_islver=0.24
_snapshot=e5a67458da39de7db8971cc5ec017fa6199a08c1
pkgrel=1
pkgdesc='The GNU Compiler Collection'
arch=(x86_64)
license=(GPL LGPL FDL custom)
url='https://gcc.gnu.org'
makedepends=(binutils libmpc gcc-ada doxygen lib32-glibc lib32-gcc-libs python git libxcrypt gdc)
checkdepends=(dejagnu inetutils)
options=(!emptydirs)
_libdir=usr/lib/gcc/$CHOST/${pkgver%%+*}
# _commit=6beb39ee6c465c21d0cc547fd66b445100cdcc35
# source=(git://gcc.gnu.org/git/gcc.git#commit=$_commit
source=(#https://sourceware.org/pub/gcc/releases/gcc-${pkgver}/gcc-${pkgver}.tar.xz{,.sig}
        gcc-$pkgver.zip::https://github.com/D-Programming-GDC/gcc/archive/$_snapshot.zip
        #http://isl.gforge.inria.fr/isl-${_islver}.tar.xz
        https://master.dl.sourceforge.net/project/libisl/isl-$_islver.tar.bz2
        c89 c99
        gdc_phobos_path.patch
        fs64270.patch
        ipa-fix-bit-CPP-when-combined-with-IPA-bit-CP.patch
        ipa-fix-ICE-in-get_default_value.patch
        gcc11-Wno-format-security.patch
)
validpgpkeys=(F3691687D867B81B51CE07D9BBE43771487328A9  # bpiotrowski@archlinux.org
              86CFFCA918CF3AF47147588051E8B148A9999C34  # evangelos@foutrelis.com
              13975A70E63C361C73AE69EF6EEB81F8981C74C7  # richard.guenther@gmail.com
              D3A93CAD751C2AF4F8C7AD516C35B99309B5FA62) # Jakub Jelinek <jakub@redhat.com>
sha256sums=('993e74560e89bd7a1ca19d52ef8b8c50d3fe2db160c275d779f3e04401627e1d'
            'fcf78dd9656c10eb8cf9fbd5f59a0b6b01386205fe1934b3b287a0a1898145c0'
            'de48736f6e4153f03d0a5d38ceb6c6fdb7f054e8f47ddd6af0a3dbf14f27b931'
            '2513c6d9984dd0a2058557bf00f06d8d5181734e41dcfe07be7ed86f2959622a'
            'c86372c207d174c0918d4aedf1cb79f7fc093649eb1ad8d9450dccc46849d308'
            '1ef190ed4562c4db8c1196952616cd201cfdd788b65f302ac2cc4dabb4d72cee'
            'fcb11c9bcea320afd202b031b48f8750aeaedaa4b0c5dddcd2c0a16381e927e4'
            '42865f2af3f48140580c4ae70b6ea03b5bdca0f29654773ef0d42ce00d60ea16'
            '504e4b5a08eb25b6c35f19fdbe0c743ae4e9015d0af4759e74150006c283585e')

if [ -n "$_snapshot" ]; then
  _basedir=gcc-$_snapshot
else
  _basedir=gcc-$pkgver
fi


prepare() {
  cd $_basedir

  # link isl for in-tree build
  ln -s ../isl-${_islver} isl

  # Do not run fixincludes
  sed -i 's@\./fixinc\.sh@-c true@' gcc/Makefile.in

  # Arch Linux installs x86_64 libraries /lib
  sed -i '/m64=/s/lib64/lib/' gcc/config/i386/t-linux64

  # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

  # configure.ac: When adding -Wno-format, also add -Wno-format-security
  patch -Np0 < "$srcdir/gcc11-Wno-format-security.patch"

  mkdir -p "$srcdir/gcc-build"
}

build() {
  cd gcc-build

  # using -pipe causes spurious test-suite failures
  # http://gcc.gnu.org/bugzilla/show_bug.cgi?id=48565
  CFLAGS=${CFLAGS/-pipe/}
  CFLAGS=${CFLAGS/-Wformat/}
  CFLAGS=${CFLAGS/-Werror=format-security/}
  CXXFLAGS=${CXXFLAGS/-pipe/}
  CXXFLAGS=${CXXFLAGS/-Wformat/}
  CXXFLAGS=${CXXFLAGS/-Werror=format-security/}

  "$srcdir/$_basedir/configure" --prefix=/usr \
      --libdir=/usr/lib \
      --libexecdir=/usr/lib \
      --mandir=/usr/share/man \
      --infodir=/usr/share/info \
      --with-bugurl=https://bugs.archlinux.org/ \
      --enable-languages=c,c++,lto,d \
      --with-isl \
      --with-linker-hash-style=gnu \
      --with-system-zlib \
      --enable-__cxa_atexit \
      --enable-cet=auto \
      --enable-checking=release \
      --enable-clocale=gnu \
      --enable-default-pie \
      --enable-default-ssp \
      --enable-gnu-indirect-function \
      --enable-gnu-unique-object \
      --enable-install-libiberty \
      --enable-linker-build-id \
      --enable-lto \
      --disable-multilib \
      --enable-plugin \
      --enable-shared \
      --enable-threads=posix \
      --enable-libphobos \
      --disable-libssp \
      --disable-libstdcxx-pch \
      --disable-libunwind-exceptions \
      --disable-werror

  make

  # make documentation
  make -C $CHOST/libstdc++-v3/doc doc-man-doxygen
}

check() {
  cd gcc-build

  # disable libphobos test to avoid segfaults and other unfunny ways to waste my time  
  sed -i '/maybe-check-target-libphobos \\/d' Makefile 

  # do not abort on error as some are "expected"
  make -k check || true
  "$srcdir/$_basedir/contrib/test_summary"
}

package_gcc-libs() {
  pkgdesc='Runtime libraries shipped by GCC'
  depends=('glibc>=2.27')
  options+=(!strip)
  provides=($pkgname-multilib libgo.so libgfortran.so libgphobos.so
            libubsan.so libasan.so libtsan.so liblsan.so)
  replaces=($pkgname-multilib libgphobos)

  cd gcc-build
  make -C $CHOST/libgcc DESTDIR="$pkgdir" install-shared
  rm -f "$pkgdir/$_libdir/libgcc_eh.a"

  for lib in libatomic \
             libgfortran \
             libgo \
             libgomp \
             libitm \
             libquadmath \
             libsanitizer/{a,l,ub,t}san \
             libstdc++-v3/src \
             libvtv; do
    make -C $CHOST/$lib DESTDIR="$pkgdir" install-toolexeclibLTLIBRARIES
  done

  make -C $CHOST/libobjc DESTDIR="$pkgdir" install-libs
  make -C $CHOST/libstdc++-v3/po DESTDIR="$pkgdir" install

  make -C $CHOST/libphobos DESTDIR="$pkgdir" install
  rm -rf "$pkgdir"/$_libdir/include/d/
  rm -f "$pkgdir"/usr/lib/libgphobos.spec

  for lib in libgomp \
             libitm \
             libquadmath; do
    make -C $CHOST/$lib DESTDIR="$pkgdir" install-info
  done

  # remove files provided by lib32-gcc-libs
  rm -rf "$pkgdir"/usr/lib32/

  # Install Runtime Library Exception
  install -Dm644 "$srcdir/$_basedir/COPYING.RUNTIME" \
    "$pkgdir/usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION"
}

package_gcc() {
  pkgdesc="The GNU Compiler Collection - C and C++ frontends"
  depends=("gcc-libs=$pkgver-$pkgrel" 'binutils>=2.28' libmpc)
  groups=('base-devel')
  optdepends=('lib32-gcc-libs: for generating code for 32-bit ABI')
  provides=($pkgname-multilib)
  replaces=($pkgname-multilib)
  options+=(staticlibs)

  cd gcc-build

  make -C gcc DESTDIR="$pkgdir" install-driver install-cpp install-gcc-ar \
    c++.install-common install-headers install-plugin install-lto-wrapper

  install -m755 -t "$pkgdir/usr/bin/" gcc/gcov{,-tool}
  install -m755 -t "$pkgdir/${_libdir}/" gcc/{cc1,cc1plus,collect2,lto1}

  make -C $CHOST/libgcc DESTDIR="$pkgdir" install
  make -C $CHOST/32/libgcc DESTDIR="$pkgdir" install
  rm -f "$pkgdir"/usr/lib{,32}/libgcc_s.so*

  make -C $CHOST/libstdc++-v3/src DESTDIR="$pkgdir" install
  make -C $CHOST/libstdc++-v3/include DESTDIR="$pkgdir" install
  make -C $CHOST/libstdc++-v3/libsupc++ DESTDIR="$pkgdir" install
  make -C $CHOST/libstdc++-v3/python DESTDIR="$pkgdir" install
  make -C $CHOST/32/libstdc++-v3/src DESTDIR="$pkgdir" install
  make -C $CHOST/32/libstdc++-v3/include DESTDIR="$pkgdir" install
  make -C $CHOST/32/libstdc++-v3/libsupc++ DESTDIR="$pkgdir" install

  make DESTDIR="$pkgdir" install-libcc1
  install -d "$pkgdir/usr/share/gdb/auto-load/usr/lib"
  mv "$pkgdir"/usr/lib/libstdc++.so.6.*-gdb.py \
    "$pkgdir/usr/share/gdb/auto-load/usr/lib/"
  rm "$pkgdir"/usr/lib{,32}/libstdc++.so*

  make DESTDIR="$pkgdir" install-fixincludes
  make -C gcc DESTDIR="$pkgdir" install-mkheaders

  make -C lto-plugin DESTDIR="$pkgdir" install
  install -dm755 "$pkgdir"/usr/lib/bfd-plugins/
  ln -s /${_libdir}/liblto_plugin.so \
    "$pkgdir/usr/lib/bfd-plugins/"

  make -C $CHOST/libgomp DESTDIR="$pkgdir" install-nodist_{libsubinclude,toolexeclib}HEADERS
  make -C $CHOST/libitm DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -C $CHOST/libquadmath DESTDIR="$pkgdir" install-nodist_libsubincludeHEADERS
  make -C $CHOST/libsanitizer DESTDIR="$pkgdir" install-nodist_{saninclude,toolexeclib}HEADERS
  make -C $CHOST/libsanitizer/asan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -C $CHOST/libsanitizer/tsan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -C $CHOST/libsanitizer/lsan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -C $CHOST/32/libgomp DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -C $CHOST/32/libitm DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -C $CHOST/32/libsanitizer DESTDIR="$pkgdir" install-nodist_{saninclude,toolexeclib}HEADERS
  make -C $CHOST/32/libsanitizer/asan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS

  make -C libiberty DESTDIR="$pkgdir" install
  install -m644 libiberty/pic/libiberty.a "$pkgdir/usr/lib"

  make -C gcc DESTDIR="$pkgdir" install-man install-info
  rm "$pkgdir"/usr/share/man/man1/{gccgo,gfortran,gdc}.1
  rm "$pkgdir"/usr/share/info/{gccgo,gfortran,gnat-style,gnat_rm,gnat_ugn,gdc}.info

  make -C libcpp DESTDIR="$pkgdir" install
  make -C gcc DESTDIR="$pkgdir" install-po

  # many packages expect this symlink
  ln -s gcc "$pkgdir"/usr/bin/cc

  # POSIX conformance launcher scripts for c89 and c99
  install -Dm755 "$srcdir/c89" "$pkgdir/usr/bin/c89"
  install -Dm755 "$srcdir/c99" "$pkgdir/usr/bin/c99"

  # install the libstdc++ man pages
  make -C $CHOST/libstdc++-v3/doc DESTDIR="$pkgdir" doc-install-man

  # remove files provided by lib32-gcc-libs
  rm -f "$pkgdir"/usr/lib32/lib{stdc++,gcc_s}.so

  # byte-compile python libraries
  python -m compileall "$pkgdir/usr/share/gcc-${pkgver%%+*}/"
  python -O -m compileall "$pkgdir/usr/share/gcc-${pkgver%%+*}/"

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION \
    "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gcc-d() {
  pkgdesc="D frontend for GCC"
  depends=("gcc=$pkgver-$pkgrel")
  provides=(gdc)
  replaces=(gdc)
  options=('staticlibs')

  cd gcc-build
  make -C gcc DESTDIR="$pkgdir" d.install-{common,man,info}

  install -Dm755 gcc/gdc "$pkgdir"/usr/bin/gdc
  install -Dm755 gcc/d21 "$pkgdir"/"$_libdir"/d21

  make -C $CHOST/libphobos DESTDIR="$pkgdir" install
  rm -f "$pkgdir/usr/lib/"lib{gphobos,gdruntime}.so*
  rm -f "$pkgdir/usr/lib32/"lib{gphobos,gdruntime}.so*

  install -d "$pkgdir"/usr/include/dlang
  ln -s /"${_libdir}"/include/d "$pkgdir"/usr/include/dlang/gdc

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION \
    "$pkgdir/usr/share/licenses/$pkgname/"
}
