# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=('llvm' 'llvm-libs' 'llvm-ocaml' 'clang' 'clang-analyzer'
         'clang-tools-extra')
pkgver=3.5.1
pkgrel=1
_ocaml_ver=4.02.1
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' "ocaml=$_ocaml_ver" 'python-sphinx' 'chrpath')
# Use gcc-multilib to build 32-bit compiler-rt libraries on x86_64 (FS#41911)
makedepends_x86_64=('gcc-multilib')
options=('staticlibs')
source=(http://llvm.org/releases/$pkgver/llvm-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/cfe-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/clang-tools-extra-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/compiler-rt-$pkgver.src.tar.xz{,.sig}
        llvm-3.5.0-force-link-pass.o.patch
        llvm-3.5.0-fix-ocaml-as-needed.patch
        llvm-Config-config.h
        llvm-Config-llvm-config.h)
sha256sums=('bf3275d2d7890015c8d8f5e6f4f882f8cf3bf51967297ebe74111d6d8b53be15'
            'SKIP'
            '6773f3f9cf815631cc7e779ec134ddd228dc8e9a250e1ea3a910610c59eb8f5c'
            'SKIP'
            'e8d011250389cfc36eb51557ca25ae66ab08173e8d53536a0747356105d72906'
            'SKIP'
            'adf4b526f33e681aff5961f0821f5b514d3fc375410008842640b56a2e6a837a'
            'SKIP'
            '5702053503d49448598eda1b8dc8c263f0df9ad7486833273e3987b5dec25a19'
            '0d8b268cf101b0a08dbe458a041c33be0500d09b1a62ed0a16f205758647a5fc'
            '312574e655f9a87784ca416949c505c452b819fad3061f2cde8aced6540a19a3'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')
validpgpkeys=('54E3BDE33185D9F69664D22455F5CD70BB5A0569'
              '11E521D646982372EB577A1F8F0871F202119294')

prepare() {
  cd "$srcdir/llvm-$pkgver.src"

  # At the present, clang must reside inside the LLVM source code tree to build
  # See http://llvm.org/bugs/show_bug.cgi?id=4840
  mv "$srcdir/cfe-$pkgver.src" tools/clang

  mv "$srcdir/clang-tools-extra-$pkgver.src" tools/clang/tools/extra

  mv "$srcdir/compiler-rt-$pkgver.src" projects/compiler-rt

  # Fix docs installation directory
  sed -i 's:$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in

  # Fix definition of LLVM_CMAKE_DIR in LLVMConfig.cmake
  sed -i '/@LLVM_CONFIG_CMAKE_DIR@/s:$(PROJ_cmake):$(PROJ_prefix)/share/llvm/cmake:' \
    cmake/modules/Makefile

  # Fix build with GCC 4.9 (patch from Debian)
  # http://llvm.org/bugs/show_bug.cgi?id=20067
  patch -Np1 -i ../llvm-3.5.0-force-link-pass.o.patch

  # Fix OCaml bindings linking with -Wl,--as-needed
  # http://llvm.org/bugs/show_bug.cgi?id=22014
  patch -Np1 -i ../llvm-3.5.0-fix-ocaml-as-needed.patch
}

build() {
  cd "$srcdir/llvm-$pkgver.src"

  # Apply strip option to configure
  _optimized_switch="enable"
  [[ $(check_option strip) == n ]] && _optimized_switch="disable"

  # Include location of libffi headers in CPPFLAGS
  CPPFLAGS+=" $(pkg-config --cflags libffi)"

  # Force the use of GCC instead of clang
  CC=gcc CXX=g++ \
  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-libffi \
    --enable-targets=all \
    --disable-expensive-checks \
    --disable-debug-runtime \
    --disable-assertions \
    --with-binutils-include=/usr/include \
    --with-python=/usr/bin/python2 \
    --$_optimized_switch-optimized

  make REQUIRES_RTTI=1
  make -C docs -f Makefile.sphinx man
  make -C docs -f Makefile.sphinx html
  make -C tools/clang/docs -f Makefile.sphinx html
}

package_llvm() {
  pkgdesc="Low Level Virtual Machine"
  depends=("llvm-libs=$pkgver-$pkgrel" 'perl')

  cd "$srcdir/llvm-$pkgver.src"

  # We move the clang directory out of the tree so it won't get installed and
  # then we bring it back in for the clang package
  mv tools/clang "$srcdir"

  # -j1 is due to race conditions during the installation of the OCaml bindings
  make -j1 DESTDIR="$pkgdir" install
  mv "$srcdir/clang" tools

  # The runtime library goes into llvm-libs
  mv -f "$pkgdir/usr/lib/libLLVM-$pkgver.so" "$srcdir/"
  mv -f "$pkgdir/usr/lib/libLLVM-${pkgver%.*}.so" "$srcdir/"

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/{ocaml,ocamldoc}
  mv "$pkgdir"/usr/{lib/ocaml,share/doc/llvm/ocamldoc} "$srcdir"

  # Remove duplicate files installed by the OCaml bindings
  rm "$pkgdir"/usr/{lib/libllvm*,share/doc/llvm/ocamldoc.tar.gz}

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Get rid of example Hello transformation
  rm "$pkgdir"/usr/lib/*LLVMHello.*

  # Symlink LLVMgold.so from /usr/lib/bfd-plugins
  # https://bugs.archlinux.org/task/28479
  install -d "$pkgdir/usr/lib/bfd-plugins"
  ln -s ../LLVMgold.so "$pkgdir/usr/lib/bfd-plugins/LLVMgold.so"

  if [[ $CARCH == x86_64 ]]; then
    # Needed for multilib (https://bugs.archlinux.org/task/29951)
    # Header stubs are taken from Fedora
    for _header in config llvm-config; do
      mv "$pkgdir/usr/include/llvm/Config/$_header"{,-64}.h
      cp "$srcdir/llvm-Config-$_header.h" \
        "$pkgdir/usr/include/llvm/Config/$_header.h"
    done
  fi

  # Install man pages
  install -d "$pkgdir/usr/share/man/man1"
  cp docs/_build/man/*.1 "$pkgdir/usr/share/man/man1/"

  # Install html docs
  cp -r docs/_build/html/* "$pkgdir/usr/share/doc/$pkgname/html/"
  rm -r "$pkgdir/usr/share/doc/$pkgname/html/_sources"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-libs() {
  pkgdesc="Low Level Virtual Machine (runtime library)"
  depends=('gcc-libs' 'zlib' 'libffi' 'ncurses')

  install -d "$pkgdir/usr/lib"
  cp -P \
    "$srcdir/libLLVM-$pkgver.so" \
    "$srcdir/libLLVM-${pkgver%.*}.so" \
    "$pkgdir/usr/lib/"

  install -Dm644 "$srcdir/llvm-$pkgver.src/LICENSE.TXT" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-ocaml() {
  pkgdesc="OCaml bindings for LLVM"
  depends=("llvm=$pkgver-$pkgrel" "ocaml=$_ocaml_ver")

  cd "$srcdir/llvm-$pkgver.src"

  install -d "$pkgdir"/{usr/lib,usr/share/doc/llvm}
  cp -r "$srcdir/ocaml" "$pkgdir/usr/lib"
  cp -r "$srcdir/ocamldoc" "$pkgdir/usr/share/doc/llvm"

  # Remove execute bit from static libraries
  chmod -x "$pkgdir"/usr/lib/ocaml/libllvm*.a

  # Remove insecure rpath
  chrpath -d "$pkgdir"/usr/lib/ocaml/*.so

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  depends=("llvm=$pkgver-$pkgrel" 'gcc')

  # Fix installation path for clang docs
  sed -i 's:$(PROJ_prefix)/share/doc/llvm:$(PROJ_prefix)/share/doc/clang:' \
    "$srcdir/llvm-$pkgver.src/Makefile.config"

  cd "$srcdir/llvm-$pkgver.src/tools/clang"

  # We move the extra tools directory out of the tree so it won't get
  # installed and then we bring it back in for the clang-tools-extra package
  mv tools/extra "$srcdir"

  make DESTDIR="$pkgdir" install
  mv "$srcdir/extra" tools/

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Revert the path change in case we want to do a repackage later
  sed -i 's:$(PROJ_prefix)/share/doc/clang:$(PROJ_prefix)/share/doc/llvm:' \
    "$srcdir/llvm-$pkgver.src/Makefile.config"

  # Install html docs
  cp -r docs/_build/html/* "$pkgdir/usr/share/doc/$pkgname/html/"
  rm -r "$pkgdir/usr/share/doc/$pkgname/html/_sources"

  # Install Python bindings
  install -d "$pkgdir/usr/lib/python2.7/site-packages"
  cp -r bindings/python/clang "$pkgdir/usr/lib/python2.7/site-packages/"
  python2 -m compileall "$pkgdir/usr/lib/python2.7/site-packages/clang"
  python2 -O -m compileall "$pkgdir/usr/lib/python2.7/site-packages/clang"

  # Install clang-format editor integration files (FS#38485)
  # Destination paths are copied from clang-format/CMakeLists.txt
  install -d "$pkgdir/usr/share/$pkgname"
  (
    cd tools/clang-format
    cp \
      clang-format-diff.py \
      clang-format-sublime.py \
      clang-format.el \
      clang-format.py \
      "$pkgdir/usr/share/$pkgname/"
    cp git-clang-format "$pkgdir/usr/bin/"
    sed -i 's|/usr/bin/python$|&2|' \
      "$pkgdir/usr/bin/git-clang-format" \
      "$pkgdir/usr/share/$pkgname/clang-format-diff.py"
  )

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
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

  # scan-build looks for clang within the same directory
  ln -s /usr/bin/clang "$pkgdir/usr/lib/clang-analyzer/scan-build/"

  # Relocate man page
  install -d "$pkgdir/usr/share/man/man1"
  mv "$pkgdir/usr/lib/clang-analyzer/scan-build/scan-build.1" \
    "$pkgdir/usr/share/man/man1/"

  # Use Python 2
  sed -i \
    -e 's|env python$|&2|' \
    -e 's|/usr/bin/python$|&2|' \
    "$pkgdir/usr/lib/clang-analyzer/scan-view/scan-view" \
    "$pkgdir/usr/lib/clang-analyzer/scan-build/set-xcode-analyzer"

  # Compile Python scripts
  python2 -m compileall "$pkgdir/usr/lib/clang-analyzer"
  python2 -O -m compileall "$pkgdir/usr/lib/clang-analyzer"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang-tools-extra() {
  pkgdesc="Extra tools built using Clang's tooling APIs"
  url="http://clang.llvm.org/"
  depends=("clang=$pkgver-$pkgrel")

  cd "$srcdir/llvm-$pkgver.src/tools/clang/tools/extra"

  make DESTDIR="$pkgdir" install

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

# vim:set ts=2 sw=2 et:
