# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=('llvm' 'llvm-libs' 'llvm-ocaml' 'lldb' 'clang' 'clang-analyzer'
         'clang-tools-extra')
pkgver=3.7.0
pkgrel=1
_ocaml_ver=4.02.3
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' "ocaml=$_ocaml_ver" 'python-sphinx' 'chrpath'
             'ocaml-ctypes' 'ocaml-findlib' 'libedit' 'swig')
# Use gcc-multilib to build 32-bit compiler-rt libraries on x86_64 (FS#41911)
makedepends_x86_64=('gcc-multilib')
options=('staticlibs')
source=(http://llvm.org/releases/$pkgver/llvm-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/cfe-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/clang-tools-extra-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/compiler-rt-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/lldb-$pkgver.src.tar.xz{,.sig}
        llvm-Config-config.h
        llvm-Config-llvm-config.h)
sha256sums=('ab45895f9dcdad1e140a3a79fd709f64b05ad7364e308c0e582c5b02e9cc3153'
            'SKIP'
            '4ed740c5a91df1c90a4118c5154851d6a475f39a91346bdf268c1c29c13aa1cc'
            'SKIP'
            '8ae8a0a3a96b7a700412d67df0af172cb2fc1326beec575fcc0f71d2e72709cd'
            'SKIP'
            '227fa998520bc94974a428dc8e7654d9bdf277e5bc70d4064ebc05691bd62b0b'
            'SKIP'
            'f4d7505bc111044eaa4033af012221e492938405b62522b8e3e354c20c4b71e9'
            'SKIP'
            '312574e655f9a87784ca416949c505c452b819fad3061f2cde8aced6540a19a3'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')
validpgpkeys=('11E521D646982372EB577A1F8F0871F202119294'
              'B6C8F98282B944E3B0D5C2530FC3042E345AD05D')

prepare() {
  cd "$srcdir/llvm-$pkgver.src"

  # At the present, clang must reside inside the LLVM source code tree to build
  # See http://llvm.org/bugs/show_bug.cgi?id=4840
  mv "$srcdir/cfe-$pkgver.src" tools/clang

  mv "$srcdir/clang-tools-extra-$pkgver.src" tools/clang/tools/extra

  mv "$srcdir/compiler-rt-$pkgver.src" projects/compiler-rt

  mv "$srcdir/lldb-$pkgver.src" tools/lldb

  # Fix compiler-rt build on i686
  # https://llvm.org/bugs/show_bug.cgi?id=22661
  sed -i '/ifeq.*CompilerTargetArch/s/i386/i686/g' \
    projects/compiler-rt/make/platform/clang_linux.mk

  # Fix docs installation directory
  sed -i 's:$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in

  # Use Python 2
  find tools/lldb -name Makefile -exec sed -i 's/python-config/python2-config/' {} +
  sed -i -e 's|env python|&2|' -e 's|/usr/bin/python$|&2|' \
    tools/lldb/scripts/Python/{build-swig-Python,finish-swig-Python-LLDB}.sh \
    tools/lldb/examples/python/symbolication.py

  mkdir build
}

build() {
  cd "$srcdir/llvm-$pkgver.src/build"

  # Include location of libffi headers in CPPFLAGS
  CPPFLAGS+=" $(pkg-config --cflags libffi)"

  # Force the use of GCC instead of clang
  CC=gcc CXX=g++ \
  ../configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-optimized \
    --enable-libffi \
    --enable-targets=all \
    --enable-bindings=ocaml \
    --disable-assertions \
    --with-binutils-include=/usr/include \
    --with-python=/usr/bin/python2

  make REQUIRES_RTTI=1
  make -C ../docs -f Makefile.sphinx man
  make -C ../docs -f Makefile.sphinx html
  make -C ../tools/clang/docs -f Makefile.sphinx man
  make -C ../tools/clang/docs -f Makefile.sphinx html
}

package_llvm() {
  pkgdesc="Low Level Virtual Machine"
  depends=("llvm-libs=$pkgver-$pkgrel" 'perl')

  cd "$srcdir/llvm-$pkgver.src"

  # Temporarily rename clang and lldb directories so they don't get installed
  for tool in clang lldb; do
      mv build/tools/$tool{,.hidden}
      mv tools/$tool{,.hidden}
  done

  make -C build DESTDIR="$pkgdir" install

  # Restore original tool directory names
  for tool in clang lldb; do
      mv build/tools/$tool{.hidden,}
      mv tools/$tool{.hidden,}
  done

  # The runtime libraries go into llvm-libs
  mv -f "$pkgdir/usr/lib/libLLVM-$pkgver.so" "$srcdir/"
  mv -f "$pkgdir/usr/lib/libLLVM-${pkgver%.*}.so" "$srcdir/"
  mv -f "$pkgdir"/usr/lib/{LLVMgold,libLTO,BugpointPasses}.so "$srcdir/"

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/{ocaml,ocamldoc}
  mv "$pkgdir"/usr/{lib/ocaml,share/doc/llvm/ocamldoc} "$srcdir"

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Get rid of example Hello transformation
  rm "$pkgdir"/usr/lib/*LLVMHello.*

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
  pkgdesc="Low Level Virtual Machine (runtime libraries)"
  depends=('gcc-libs' 'zlib' 'libffi' 'libedit' 'ncurses')

  install -d "$pkgdir/usr/lib"
  cp -P \
    "$srcdir/libLLVM-$pkgver.so" \
    "$srcdir/libLLVM-${pkgver%.*}.so" \
    "$srcdir"/{LLVMgold,libLTO,BugpointPasses}.so \
    "$pkgdir/usr/lib/"

  # Symlink LLVMgold.so from /usr/lib/bfd-plugins
  # https://bugs.archlinux.org/task/28479
  install -d "$pkgdir/usr/lib/bfd-plugins"
  ln -s ../LLVMgold.so "$pkgdir/usr/lib/bfd-plugins/LLVMgold.so"

  install -Dm644 "$srcdir/llvm-$pkgver.src/LICENSE.TXT" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-ocaml() {
  pkgdesc="OCaml bindings for LLVM"
  depends=("llvm=$pkgver-$pkgrel" "ocaml=$_ocaml_ver" 'ocaml-ctypes')

  cd "$srcdir/llvm-$pkgver.src"

  install -d "$pkgdir"/{usr/lib,usr/share/doc/llvm}
  cp -r "$srcdir/ocaml" "$pkgdir/usr/lib"
  cp -r "$srcdir/ocamldoc" "$pkgdir/usr/share/doc/llvm"

  # Remove execute bit from static libraries
  chmod -x "$pkgdir"/usr/lib/ocaml/libllvm*.a

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lldb() {
  pkgdesc="Next generation, high-performance debugger"
  url="http://lldb.llvm.org/"
  depends=("llvm-libs=$pkgver-$pkgrel" 'libedit' 'python2')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/lldb DESTDIR="$pkgdir" install

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Remove insecure rpath
  chrpath -d "$pkgdir"/usr/bin/*

  # Compile Python scripts
  python2 -m compileall "$pkgdir/usr/lib/python2.7/site-packages/lldb"
  python2 -O -m compileall "$pkgdir/usr/lib/python2.7/site-packages/lldb"

  install -Dm644 tools/lldb/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  depends=("llvm-libs=$pkgver-$pkgrel" 'gcc')

  cd "$srcdir/llvm-$pkgver.src"

  # Fix installation path for clang docs
  sed -i 's:$(PROJ_prefix)/share/doc/llvm:$(PROJ_prefix)/share/doc/clang:' \
    build/Makefile.config

  # Temporarily rename clang extra tools directory so they don't get installed
  mv build/tools/clang/tools/extra{,.hidden}
  mv tools/clang/tools/extra{,.hidden}

  make -C build/tools/clang DESTDIR="$pkgdir" install

  # Restore original extra tools directory name
  mv build/tools/clang/tools/extra{.hidden,}
  mv tools/clang/tools/extra{.hidden,}

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Revert the path change in case we want to do a repackage later
  sed -i 's:$(PROJ_prefix)/share/doc/clang:$(PROJ_prefix)/share/doc/llvm:' \
    build/Makefile.config

  # Install man pages
  install -d "$pkgdir/usr/share/man/man1"
  cp tools/clang/docs/_build/man/*.1 "$pkgdir/usr/share/man/man1/"

  # Install html docs
  cp -r tools/clang/docs/_build/html/* "$pkgdir/usr/share/doc/$pkgname/html/"
  rm -r "$pkgdir/usr/share/doc/$pkgname/html/_sources"

  # Install Python bindings
  install -d "$pkgdir/usr/lib/python2.7/site-packages"
  cp -r tools/clang/bindings/python/clang "$pkgdir/usr/lib/python2.7/site-packages/"
  python2 -m compileall "$pkgdir/usr/lib/python2.7/site-packages/clang"
  python2 -O -m compileall "$pkgdir/usr/lib/python2.7/site-packages/clang"

  # Install clang-format editor integration files (FS#38485)
  # Destination paths are copied from clang-format/CMakeLists.txt
  install -d "$pkgdir/usr/share/$pkgname"
  (
    cd tools/clang/tools/clang-format
    cp \
      clang-format-diff.py \
      clang-format-sublime.py \
      clang-format.el \
      clang-format.py \
      "$pkgdir/usr/share/$pkgname/"
    cp git-clang-format "$pkgdir/usr/bin/"
    sed -i 's|/usr/bin/env python|&2|' \
      "$pkgdir/usr/bin/git-clang-format" \
      "$pkgdir/usr/share/$pkgname/clang-format-diff.py"
  )

  install -Dm644 tools/clang/LICENSE.TXT \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
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
  sed -i -e 's|env python$|&2|' -e 's|/usr/bin/python$|&2|' \
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

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/clang/tools/extra DESTDIR="$pkgdir" install

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  install -Dm644 tools/clang/tools/extra/LICENSE.TXT \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

# vim:set ts=2 sw=2 et:
