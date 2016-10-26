# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=('llvm' 'llvm-libs' 'llvm-ocaml' 'lldb' 'clang' 'clang-tools-extra')
pkgver=3.9.0
pkgrel=1
_ocaml_ver=4.03.0
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('cmake' 'libffi' 'python2' "ocaml=$_ocaml_ver" 'python-sphinx'
             'ocaml-ctypes' 'ocaml-findlib' 'libedit' 'swig')
# Use gcc-multilib to build 32-bit compiler-rt libraries on x86_64 (FS#41911)
makedepends_x86_64=('gcc-multilib')
options=('staticlibs')
source=(http://llvm.org/releases/$pkgver/llvm-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/cfe-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/clang-tools-extra-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/compiler-rt-$pkgver.src.tar.xz{,.sig}
        http://llvm.org/releases/$pkgver/lldb-$pkgver.src.tar.xz{,.sig}
        msan-prevent-initialization-failure-with-newer-glibc.patch
        llvm-Config-llvm-config.h)
sha256sums=('66c73179da42cee1386371641241f79ded250e117a79f571bbd69e56daa48948'
            'SKIP'
            '7596a7c7d9376d0c89e60028fe1ceb4d3e535e8ea8b89e0eb094e0dcb3183d28'
            'SKIP'
            '5b7aec46ec8e999ec683c87ad744082e1133781ee4b01905b4bdae5d20785f14'
            'SKIP'
            'e0e5224fcd5740b61e416c549dd3dcda92f10c524216c1edb5e979e42078a59a'
            'SKIP'
            '61280e07411e3f2b4cca0067412b39c16b0a9edd19d304d3fc90249899d12384'
            'SKIP'
            '8e4f194c2283b91644a7fff43bc4e58c36b5507f2a4d90b72f275c0bd7511c20'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')
validpgpkeys=('B6C8F98282B944E3B0D5C2530FC3042E345AD05D')

prepare() {
  cd "$srcdir/llvm-$pkgver.src"

  # At the present, clang must reside inside the LLVM source code tree to build
  # See http://llvm.org/bugs/show_bug.cgi?id=4840
  mv "$srcdir/cfe-$pkgver.src" tools/clang

  mv "$srcdir/clang-tools-extra-$pkgver.src" tools/clang/tools/extra

  mv "$srcdir/compiler-rt-$pkgver.src" projects/compiler-rt

  mv "$srcdir/lldb-$pkgver.src" tools/lldb

  # https://reviews.llvm.org/D24736
  patch -Np0 -d projects/compiler-rt <../msan-prevent-initialization-failure-with-newer-glibc.patch

  mkdir build
}

build() {
  cd "$srcdir/llvm-$pkgver.src/build"

  cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DLLVM_BUILD_LLVM_DYLIB=ON \
    -DLLVM_LINK_LLVM_DYLIB=ON \
    -DLLVM_ENABLE_RTTI=ON \
    -DLLVM_ENABLE_FFI=ON \
    -DLLVM_BUILD_TESTS=ON \
    -DLLVM_BUILD_DOCS=ON \
    -DLLVM_ENABLE_SPHINX=ON \
    -DLLVM_ENABLE_DOXYGEN=OFF \
    -DLLDB_DISABLE_LIBEDIT=1 \
    -DSPHINX_WARNINGS_AS_ERRORS=OFF \
    -DFFI_INCLUDE_DIR=$(pkg-config --variable=includedir libffi) \
    -DLLVM_BINUTILS_INCDIR=/usr/include \
    ..

  make
  make ocaml_doc

  # Disable automatic installation of components that go into subpackages
  sed -i '/\(clang\|lldb\)\/cmake_install.cmake/d' tools/cmake_install.cmake
  sed -i '/extra\/cmake_install.cmake/d' tools/clang/tools/cmake_install.cmake
  sed -i '/compiler-rt\/cmake_install.cmake/d' projects/cmake_install.cmake
}

check() {
  cd "$srcdir/llvm-$pkgver.src/build"
  make check
  make check-clang
}

package_llvm() {
  pkgdesc="Low Level Virtual Machine"
  depends=("llvm-libs=$pkgver-$pkgrel" 'perl')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build DESTDIR="$pkgdir" install

  # Remove documentation sources
  rm -r "$pkgdir"/usr/share/doc/$pkgname/html/{_sources,.buildinfo}

  # The runtime libraries go into llvm-libs
  mv -f "$pkgdir"/usr/lib/lib{LLVM,LTO}*.so "$srcdir"
  mv -f "$pkgdir"/usr/lib/LLVMgold.so "$srcdir"

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/ocaml.{lib,doc}
  mv "$pkgdir/usr/lib/ocaml" "$srcdir/ocaml.lib"
  mv "$pkgdir/usr/docs/ocaml/html" "$srcdir/ocaml.doc"
  rm -r "$pkgdir/usr/docs"

  if [[ $CARCH == x86_64 ]]; then
    # Needed for multilib (https://bugs.archlinux.org/task/29951)
    # Header stub is taken from Fedora
    mv "$pkgdir/usr/include/llvm/Config/llvm-config"{,-64}.h
    cp "$srcdir/llvm-Config-llvm-config.h" \
      "$pkgdir/usr/include/llvm/Config/llvm-config.h"
  fi

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-libs() {
  pkgdesc="Low Level Virtual Machine (runtime libraries)"
  depends=('gcc-libs' 'zlib' 'libffi' 'libedit' 'ncurses')

  install -d "$pkgdir/usr/lib"
  cp -P \
    "$srcdir"/lib{LLVM,LTO}*.so \
    "$srcdir"/LLVMgold.so \
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

  install -d "$pkgdir"/{usr/lib,usr/share/doc}
  cp -a "$srcdir/ocaml.lib" "$pkgdir/usr/lib/ocaml"
  cp -a "$srcdir/ocaml.doc" "$pkgdir/usr/share/doc/$pkgname"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lldb() {
  pkgdesc="Next generation, high-performance debugger"
  url="http://lldb.llvm.org/"
  depends=('libxml2' 'python2' 'python2-six')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/lldb DESTDIR="$pkgdir" install

  # https://bugs.archlinux.org/task/50759
  sed -i "/import_module('_lldb')/s/_lldb/lldb.&/" \
  "$pkgdir/usr/lib/python2.7/site-packages/lldb/__init__.py"

  # Remove bundled six library
  rm "$pkgdir/usr/lib/python2.7/site-packages/six.py"

  # Compile Python scripts
  python2 -m compileall "$pkgdir/usr/lib/python2.7/site-packages/lldb"
  python2 -O -m compileall "$pkgdir/usr/lib/python2.7/site-packages/lldb"

  install -Dm644 tools/lldb/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  depends=("llvm-libs=$pkgver-$pkgrel" 'gcc' 'libxml2')
  optdepends=('openmp: OpenMP support in clang with -fopenmp'
              'python2: for scan-view and git-clang-format')
  provides=("clang-analyzer=$pkgver")
  conflicts=('clang-analyzer')
  replaces=('clang-analyzer')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/clang DESTDIR="$pkgdir" install
  make -C build/projects/compiler-rt DESTDIR="$pkgdir" install

  # Remove documentation sources
  rm -r "$pkgdir"/usr/share/doc/$pkgname/html/{_sources,.buildinfo}

  # Move analyzer scripts out of /usr/libexec
  mv "$pkgdir"/usr/libexec/{ccc,c++}-analyzer "$pkgdir/usr/lib/clang/"
  rmdir "$pkgdir/usr/libexec"
  sed -i 's|libexec|lib/clang|' "$pkgdir/usr/bin/scan-build"

  # Install Python bindings
  install -d "$pkgdir/usr/lib/python2.7/site-packages"
  cp -a tools/clang/bindings/python/clang "$pkgdir/usr/lib/python2.7/site-packages/"

  # Use Python 2
  sed -i 's|/usr/bin/env python|&2|' \
    "$pkgdir/usr/bin/scan-view" \
    "$pkgdir/usr/bin/git-clang-format" \
    "$pkgdir/usr/share/$pkgname/clang-format-diff.py"

  # Compile Python scripts
  python2 -m compileall "$pkgdir"
  python2 -O -m compileall "$pkgdir"

  install -Dm644 tools/clang/LICENSE.TXT \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang-tools-extra() {
  pkgdesc="Extra tools built using clang's tooling APIs"
  url="http://clang.llvm.org/"
  depends=("clang=$pkgver-$pkgrel")

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/clang/tools/extra DESTDIR="$pkgdir" install

  # Use Python 2
  sed -i 's|/usr/bin/env python|&2|' \
    "$pkgdir"/usr/share/clang/{clang-tidy-diff,run-clang-tidy,run-find-all-symbols}.py

  install -Dm644 tools/clang/tools/extra/LICENSE.TXT \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

# vim:set ts=2 sw=2 et:
