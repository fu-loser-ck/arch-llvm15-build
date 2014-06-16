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
pkgver=3.4.2
_base_ver=3.4
pkgrel=1
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' 'ocaml' 'python-sphinx')
options=('staticlibs')
source=(http://llvm.org/releases/$pkgver/llvm-$pkgver.src.tar.gz{,.sig}
        http://llvm.org/releases/$pkgver/cfe-$pkgver.src.tar.gz{,.sig}
        http://llvm.org/releases/$_base_ver/clang-tools-extra-$_base_ver.src.tar.gz{,.sig}
        http://llvm.org/releases/$_base_ver/compiler-rt-$_base_ver.src.tar.gz{,.sig}
        clang-3.3-use-gold-linker.patch
        clang-3.4-fstack-protector-strong.patch
        llvm-3.4-provide-cmake-modules.patch
        llvm-Config-config.h
        llvm-Config-llvm-config.h)
sha256sums=('17038d47069ad0700c063caed76f0c7259628b0e79651ce2b540d506f2f1efd7'
            'SKIP'
            '5ba6f5772f8d00f445209356a7daf83c5bca2da5acd10de517ad2359ae95bc10'
            'SKIP'
            'ba85187551ae97fe1c8ab569903beae5ff0900e21233e5eb5389f6ceab1028b4'
            'SKIP'
            'f37c89b1383ce462d47537a0245ac798600887a9be9f63073e16b79ed536ab5c'
            'SKIP'
            '8240adda155d7961eeb5d07ed50ead10cb7125f70283dff7f1c9fee9df3cea09'
            '7a2a1ddc94f67e643c1ab74601ec07deb6d5d344d4b19ed17c900afb2f6f2863'
            'b6bb154d5ec998328e818bb09acfc6229e41367ba45cea7cc5b2dd2a7c835cf5'
            '312574e655f9a87784ca416949c505c452b819fad3061f2cde8aced6540a19a3'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')

prepare() {
  # Change directory names to release names so we don't need to change the
  # whole PKGBUILD
  mv llvm-$pkgver{.src,}
  mv cfe-$pkgver.src clang-$pkgver

  cd "$srcdir/llvm-$pkgver"

  # At the present, clang must reside inside the LLVM source code tree to build
  # See http://llvm.org/bugs/show_bug.cgi?id=4840
  mv "$srcdir/clang-$pkgver" tools/clang

  mv "$srcdir/clang-tools-extra-$_base_ver" tools/clang/tools/extra

  mv "$srcdir/compiler-rt-$_base_ver" projects/compiler-rt

  # Fix docs installation directory
  sed -i 's:\$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in

  # Make -flto work; use ld.gold instead of the default linker
  patch -d tools/clang -Np1 -i "$srcdir/clang-3.3-use-gold-linker.patch"

  # Add command line option -fstack-protector-strong
  # http://reviews.llvm.org/rL201120
  patch -d tools/clang -Np0 -i "$srcdir/clang-3.4-fstack-protector-strong.patch"

  # Provide CMake modules (FS#38705)
  # http://reviews.llvm.org/rL201047
  # http://reviews.llvm.org/rL201048
  # http://reviews.llvm.org/rL201053
  patch -Np0 -i "$srcdir/llvm-3.4-provide-cmake-modules.patch"
}

build() {
  cd "$srcdir/llvm-$pkgver"

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

  cd "$srcdir/llvm-$pkgver"

  # We move the clang directory out of the tree so it won't get installed and
  # then we bring it back in for the clang package
  mv tools/clang "$srcdir"

  # -j1 is due to race conditions during the installation of the OCaml bindings
  make -j1 DESTDIR="$pkgdir" install
  mv "$srcdir/clang" tools

  # The runtime library goes into llvm-libs
  mv "$pkgdir/usr/lib/libLLVM-$pkgver.so" "$srcdir/"
  mv "$pkgdir/usr/lib/libLLVM-$_base_ver.so" "$srcdir/"

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/{ocaml,ocamldoc}
  mv "$pkgdir"/usr/{lib/ocaml,share/doc/llvm/ocamldoc} "$srcdir"

  # Remove duplicate files installed by the OCaml bindings
  rm "$pkgdir"/usr/{lib/libllvm*,share/doc/llvm/ocamldoc.tar.gz}

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Get rid of example Hello transformation
  rm "$pkgdir"/usr/lib/*LLVMHello.*

  # Symlink LLVMgold.so into /usr/lib/bfd-plugins
  # (https://bugs.archlinux.org/task/28479)
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
    "$srcdir/libLLVM-$_base_ver.so" \
    "$pkgdir/usr/lib/"

  install -Dm644 "$srcdir/llvm-$pkgver/LICENSE.TXT" \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-ocaml() {
  pkgdesc="OCaml bindings for LLVM"
  depends=("llvm=$pkgver-$pkgrel" 'ocaml')

  cd "$srcdir/llvm-$pkgver"

  install -d "$pkgdir"/{usr/lib,usr/share/doc/llvm}
  cp -r "$srcdir/ocaml" "$pkgdir/usr/lib"
  cp -r "$srcdir/ocamldoc" "$pkgdir/usr/share/doc/llvm"

  # Remove execute bit from static libraries
  chmod -x "$pkgdir"/usr/lib/ocaml/libllvm*.a

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  depends=("llvm=$pkgver-$pkgrel" 'gcc')

  # Fix installation path for clang docs
  sed -i 's:$(PROJ_prefix)/share/doc/llvm:$(PROJ_prefix)/share/doc/clang:' \
    "$srcdir/llvm-$pkgver/Makefile.config"

  cd "$srcdir/llvm-$pkgver/tools/clang"

  # We move the extra tools directory out of the tree so it won't get
  # installed and then we bring it back in for the clang-tools-extra package
  mv tools/extra "$srcdir"

  make DESTDIR="$pkgdir" install
  mv "$srcdir/extra" tools/

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  # Revert the path change in case we want to do a repackage later
  sed -i 's:$(PROJ_prefix)/share/doc/clang:$(PROJ_prefix)/share/doc/llvm:' \
    "$srcdir/llvm-$pkgver/Makefile.config"

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
    sed -i 's|/usr/bin/python$|&2|' "$pkgdir/usr/bin/git-clang-format"
  )

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang-analyzer() {
  pkgdesc="A source code analysis framework"
  url="http://clang-analyzer.llvm.org/"
  depends=("clang=$pkgver-$pkgrel" 'python2')

  cd "$srcdir/llvm-$pkgver/tools/clang"

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

  cd "$srcdir/llvm-$pkgver/tools/clang/tools/extra"

  make DESTDIR="$pkgdir" install

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/*.a

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

# vim:set ts=2 sw=2 et:
