# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=('llvm' 'llvm-ocaml' 'clang' 'clang-analyzer')
pkgver=3.1
pkgrel=5
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' 'ocaml')
source=(http://llvm.org/releases/$pkgver/$pkgname-$pkgver.src.tar.gz
        http://llvm.org/releases/$pkgver/clang-$pkgver.src.tar.gz
        http://dev.archlinux.org/~foutrelis/sources/compiler-rt/compiler-rt-$pkgver.src.tar.xz
        llvm-Config-config.h
        llvm-Config-llvm-config.h
        cindexer-clang-path.patch
        clang-pure64.patch
        enable-lto.patch
        llvm-3.1-fix-debug-line-info.patch
        clang-3.1-fix-libprofile_rt.a-location.patch)
sha256sums=('1ea05135197b5400c1f88d00ff280d775ce778f8f9ea042e25a1e1e734a4b9ab'
            'ff63e215dcd3e2838ffdea38502f8d35bab17e487f3c3799579961e452d5a786'
            '563d8a5ef86123ed8775e115ad7f90c1aa3e80f70b4e587f1bccab2c10753558'
            '312574e655f9a87784ca416949c505c452b819fad3061f2cde8aced6540a19a3'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48'
            '3074df5322900e087377a8e03a02115463ccc0011c25917c2f06df11facd9b92'
            '288a82fbff17bc554f5863734246500e637882af33ee8511019d5e0d6cd20524'
            'f7145e203ffb4ce2c01976027f7840a9520e5341a9945f2459b6b11e5422d5b7'
            'db1f1aadebbc4c4232bdad49fb9b7dc61eac727085c63154b870fa9ce64fd18d'
            '0d32ad283566357ca1bfbeb4cbe6b0b961943b79d3d718ed0435101c05629137')

build() {
  cd "$srcdir/$pkgname-$pkgver.src"

  # At the present, clang must reside inside the LLVM source code tree to build
  # See http://llvm.org/bugs/show_bug.cgi?id=4840
  rm -rf tools/clang
  cp -r "$srcdir/clang-$pkgver.src" tools/clang

  rm -rf projects/compiler-rt
  cp -r "$srcdir/compiler-rt-$pkgver.src" projects/compiler-rt

  # Fix symbolic links from OCaml bindings to LLVM libraries
  sed -i 's:\$(PROJ_libdir):/usr/lib/llvm:' bindings/ocaml/Makefile.ocaml

  # Fix installation directories, ./configure doesn't seem to set them right
  sed -i -e 's:\$(PROJ_prefix)/etc/llvm:/etc/llvm:' \
         -e 's:\$(PROJ_prefix)/lib:$(PROJ_prefix)/lib/llvm:' \
         -e 's:\$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in
  sed -i '/ActiveLibDir = ActivePrefix/s:lib:lib/llvm:' \
    tools/llvm-config/llvm-config.cpp
  sed -i 's:LLVM_LIBDIR="${prefix}/lib":LLVM_LIBDIR="${prefix}/lib/llvm":' \
    autoconf/configure.ac \
    configure

  # Fix insecure rpath (http://bugs.archlinux.org/task/14017)
  sed -i 's:$(RPATH) -Wl,$(\(ToolDir\|LibDir\|ExmplDir\))::g' Makefile.rules

  # Fix clang path in CIndexer.cpp (https://bugs.archlinux.org/task/22799)
  patch -d tools/clang -Np0 -i "$srcdir/cindexer-clang-path.patch"

  if [[ $CARCH == x86_64 ]]; then
    # Adjust linker path
    patch -d tools/clang -Np0 -i "$srcdir/clang-pure64.patch"
  fi

  # Make -flto work
  # Use gold instead of default linker, and always use the plugin
  patch -d tools/clang -Np0 -i "$srcdir/enable-lto.patch"

  # Fix FS#29984: [clang] -coverage is broken
  patch -d tools/clang -Np1 -i \
    "$srcdir/clang-3.1-fix-libprofile_rt.a-location.patch"

  # Fix FS#31098: LLVM 3.1 produces invalid debug information
  # http://llvm.org/bugs/show_bug.cgi?id=13211
  patch -Np1 -i "$srcdir/llvm-3.1-fix-debug-line-info.patch"

  # Apply strip option to configure
  _optimized_switch="enable"
  [[ $(check_option strip) == n ]] && _optimized_switch="disable"

  # Include location of libffi headers in CPPFLAGS
  export CPPFLAGS="$CPPFLAGS $(pkg-config --cflags libffi)"

  # Use Python 2
  mkdir "$srcdir/python2-path"
  ln -s /usr/bin/python2 "$srcdir/python2-path/python"
  export PATH="$srcdir/python2-path:$PATH"

  # Force the use of GCC instead of clang
  CC=gcc CXX=g++ \
  ./configure \
    --prefix=/usr \
    --libdir=/usr/lib/llvm \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-libffi \
    --enable-targets=all \
    --disable-expensive-checks \
    --disable-debug-runtime \
    --disable-assertions \
    --with-binutils-include=/usr/include \
    --$_optimized_switch-optimized

  make REQUIRES_RTTI=1
}

package_llvm() {
  pkgdesc="Low Level Virtual Machine"
  depends=('perl' 'libffi')

  cd "$srcdir/$pkgname-$pkgver.src"

  # We move the clang directory out of the tree so it won't get installed and
  # then we bring it back in for the clang package
  mv tools/clang "$srcdir"

  # -j1 is due to race conditions during the installation of the OCaml bindings
  make -j1 DESTDIR="$pkgdir" install
  mv "$srcdir/clang" tools

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/{ocaml,ocamldoc}
  mv "$pkgdir"/usr/{lib/ocaml,share/doc/llvm/ocamldoc} "$srcdir"

  # Remove duplicate files installed by the OCaml bindings
  rm "$pkgdir"/usr/{lib/llvm/libllvm*,share/doc/llvm/ocamldoc.tar.gz}

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/llvm/*.a

  # Get rid of example Hello transformation
  rm "$pkgdir"/usr/lib/llvm/*LLVMHello.*

  # Add ld.so.conf.d entry
  install -d "$pkgdir/etc/ld.so.conf.d"
  echo /usr/lib/llvm >"$pkgdir/etc/ld.so.conf.d/llvm.conf"

  # Symlink LLVMgold.so into /usr/lib/bfd-plugins
  # (https://bugs.archlinux.org/task/28479)
  install -d "$pkgdir/usr/lib/bfd-plugins"
  ln -s ../llvm/LLVMgold.so "$pkgdir/usr/lib/bfd-plugins/LLVMgold.so"

  if [[ $CARCH == x86_64 ]]; then
    # Needed for multilib (https://bugs.archlinux.org/task/29951)
    # Header stubs are taken from Fedora
    for _header in config llvm-config; do
      mv "$pkgdir/usr/include/llvm/Config/$_header"{,-64}.h
      cp "$srcdir/llvm-Config-$_header.h" \
        "$pkgdir/usr/include/llvm/Config/$_header.h"
    done
  fi

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-ocaml() {
  pkgdesc="OCaml bindings for LLVM"
  depends=("llvm=$pkgver-$pkgrel" 'ocaml')

  cd "$srcdir/llvm-$pkgver.src"

  install -d "$pkgdir"/{usr/lib,usr/share/doc/llvm}
  cp -r "$srcdir/ocaml" "$pkgdir/usr/lib"
  cp -r "$srcdir/ocamldoc" "$pkgdir/usr/share/doc/llvm"

  # Remove execute bit from static libraries
  chmod -x "$pkgdir"/usr/lib/ocaml/libllvm*.a

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/llvm-ocaml/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  depends=("llvm=$pkgver-$pkgrel" 'gcc')

  # Fix installation path for clang docs
  sed -i 's:$(PROJ_prefix)/share/doc/llvm:$(PROJ_prefix)/share/doc/clang:' \
    "$srcdir/llvm-$pkgver.src/Makefile.config"

  cd "$srcdir/llvm-$pkgver.src/tools/clang"
  make DESTDIR="$pkgdir" install

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/llvm/*.a

  # Revert the path change in case we want to do a repackage later
  sed -i 's:$(PROJ_prefix)/share/doc/clang:$(PROJ_prefix)/share/doc/llvm:' \
    "$srcdir/llvm-$pkgver.src/Makefile.config"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/clang/LICENSE"
}

package_clang-analyzer() {
  pkgdesc="A source code analysis framework"
  url="http://clang-analyzer.llvm.org/"
  depends=("clang=$pkgver-$pkgrel" 'python2')

  cd "$srcdir/llvm-$pkgver.src/tools/clang"

  install -d "$pkgdir"/usr/{bin,lib/clang-analyzer}
  for _tool in scan-{build,view}; do
    cp -r tools/$_tool "$pkgdir/usr/lib/clang-analyzer/"
    ln -s /usr/lib/clang-analyzer/$_tool/$_tool "$pkgdir/usr/bin/"
  done

  # Use Python 2
  sed -i \
    -e 's|env python$|&2|' \
    -e 's|/usr/bin/python$|&2|' \
    "$pkgdir/usr/lib/clang-analyzer/scan-view/scan-view" \
    "$pkgdir/usr/lib/clang-analyzer/scan-build/set-xcode-analyzer"

  # Compile Python scripts
  python2 -m compileall "$pkgdir/usr/lib/clang-analyzer"
  python2 -O -m compileall "$pkgdir/usr/lib/clang-analyzer"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/clang-analyzer/LICENSE"
}

# vim:set ts=2 sw=2 et:
