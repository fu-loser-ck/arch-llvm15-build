# Maintainer: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=('llvm' 'llvm-libs' 'llvm-ocaml' 'lld' 'lldb' 'clang' 'clang-tools-extra')
pkgver=4.0.1
pkgrel=5
_ocaml_ver=4.05.0
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('cmake' 'libffi' 'python2' "ocaml=$_ocaml_ver" 'python-sphinx'
             'ocaml-ctypes' 'ocaml-findlib' 'libedit' 'swig')
# Use gcc-multilib to build 32-bit compiler-rt libraries on x86_64 (FS#41911)
makedepends_x86_64=('gcc-multilib')
options=('staticlibs')
source=(https://releases.llvm.org/$pkgver/llvm-$pkgver.src.tar.xz{,.sig}
        https://releases.llvm.org/$pkgver/cfe-$pkgver.src.tar.xz{,.sig}
        https://releases.llvm.org/$pkgver/clang-tools-extra-$pkgver.src.tar.xz{,.sig}
        https://releases.llvm.org/$pkgver/compiler-rt-$pkgver.src.tar.xz{,.sig}
        https://releases.llvm.org/$pkgver/lld-$pkgver.src.tar.xz{,.sig}
        https://releases.llvm.org/$pkgver/lldb-$pkgver.src.tar.xz{,.sig}
        0001-GCC-compatibility-Ignore-the-fno-plt-flag.patch
        0002-Enable-SSP-and-PIE-by-default.patch
        disable-llvm-symbolizer-test.patch
        lldb-gcc7.patch
        lldb-libedit.patch
        llvm-config.h)
sha256sums=('da783db1f82d516791179fe103c71706046561f7972b18f0049242dee6712b51'
            'SKIP'
            '61738a735852c23c3bdbe52d035488cdb2083013f384d67c1ba36fabebd8769b'
            'SKIP'
            '35d1e64efc108076acbe7392566a52c35df9ec19778eb9eb12245fc7d8b915b6'
            'SKIP'
            'a3c87794334887b93b7a766c507244a7cdcce1d48b2e9249fc9a94f2c3beb440'
            'SKIP'
            '63ce10e533276ca353941ce5ab5cc8e8dcd99dbdd9c4fa49f344a212f29d36ed'
            'SKIP'
            '8432d2dfd86044a0fc21713e0b5c1d98e1d8aad863cf67562879f47f841ac47b'
            'SKIP'
            'd26239d03983fab1f34970b94d727379ca0be689f5826e50503c4d2f314f207a'
            '686b2e05a151c9a4b36d5090ddc986432ad746bc6060a00928ee3b52d6b3e723'
            '6fff47ab5ede79d45fe64bb4903b7dfc27212a38e6cd5d01e60ebd24b7557359'
            '10cca2f593c711b1b547f479f9f783ab88f9a64b356519d9aa1367e0ff6da73a'
            'b80bda6dc26792e499b3150e13c3017be4a65280b4b9f5c9f4c07b55a46d93b6'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48')
validpgpkeys=('B6C8F98282B944E3B0D5C2530FC3042E345AD05D'
              '11E521D646982372EB577A1F8F0871F202119294')

prepare() {
  cd "$srcdir/llvm-$pkgver.src"
  mkdir build

  mv "$srcdir/cfe-$pkgver.src" tools/clang
  mv "$srcdir/clang-tools-extra-$pkgver.src" tools/clang/tools/extra
  mv "$srcdir/compiler-rt-$pkgver.src" projects/compiler-rt
  mv "$srcdir/lld-$pkgver.src" tools/lld
  mv "$srcdir/lldb-$pkgver.src" tools/lldb

  # Disable test that fails when compiled as PIE
  # https://bugs.llvm.org/show_bug.cgi?id=31870
  patch -Np1 <../disable-llvm-symbolizer-test.patch

  # Enable SSP and PIE by default
  patch -Np1 -d tools/clang <../0001-GCC-compatibility-Ignore-the-fno-plt-flag.patch
  patch -Np1 -d tools/clang <../0002-Enable-SSP-and-PIE-by-default.patch

  # Fix LLDB input with recent libedit versions
  patch -Np0 -d tools/lldb <../lldb-gcc7.patch
  patch -Np1 -d tools/lldb <../lldb-libedit.patch
}

build() {
  cd "$srcdir/llvm-$pkgver.src/build"

  cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DLLVM_BUILD_LLVM_DYLIB=ON \
    -DLLVM_LINK_LLVM_DYLIB=ON \
    -DLLVM_INSTALL_UTILS=ON \
    -DLLVM_ENABLE_RTTI=ON \
    -DLLVM_ENABLE_FFI=ON \
    -DLLVM_BUILD_TESTS=ON \
    -DLLVM_BUILD_DOCS=ON \
    -DLLVM_ENABLE_SPHINX=ON \
    -DLLVM_ENABLE_DOXYGEN=OFF \
    -DSPHINX_WARNINGS_AS_ERRORS=OFF \
    -DFFI_INCLUDE_DIR=$(pkg-config --variable=includedir libffi) \
    -DLLVM_BINUTILS_INCDIR=/usr/include \
    ..

  make
  make ocaml_doc

  # Disable automatic installation of components that go into subpackages
  sed -i '/\(clang\|lld\|lldb\)\/cmake_install.cmake/d' tools/cmake_install.cmake
  sed -i '/extra\/cmake_install.cmake/d' tools/clang/tools/cmake_install.cmake
  sed -i '/compiler-rt\/cmake_install.cmake/d' projects/cmake_install.cmake
}

check() {
  cd "$srcdir/llvm-$pkgver.src/build"
  make check-{llvm,clang,clang-tools,lld}
}

package_llvm() {
  pkgdesc="Low Level Virtual Machine"
  depends=('llvm-libs' 'perl')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build DESTDIR="$pkgdir" install

  # Remove documentation sources
  rm -r "$pkgdir"/usr/share/doc/$pkgname/html/{_sources,.buildinfo}

  # The runtime libraries go into llvm-libs
  mv -f "$pkgdir"/usr/lib/lib{LLVM,LTO}*.so* "$srcdir"
  mv -f "$pkgdir"/usr/lib/LLVMgold.so "$srcdir"

  # OCaml bindings go to a separate package
  rm -rf "$srcdir"/ocaml.{lib,doc}
  mv "$pkgdir/usr/lib/ocaml" "$srcdir/ocaml.lib"
  mv "$pkgdir/usr/share/doc/$pkgname/ocaml-html" "$srcdir/ocaml.doc"

  if [[ $CARCH == x86_64 ]]; then
    # Needed for multilib (https://bugs.archlinux.org/task/29951)
    # Header stub is taken from Fedora
    mv "$pkgdir/usr/include/llvm/Config/llvm-config"{,-64}.h
    cp "$srcdir/llvm-config.h" "$pkgdir/usr/include/llvm/Config/llvm-config.h"
  fi

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_llvm-libs() {
  pkgdesc="Low Level Virtual Machine (runtime libraries)"
  depends=('gcc-libs' 'zlib' 'libffi' 'libedit' 'ncurses')

  install -d "$pkgdir/usr/lib"
  cp -P \
    "$srcdir"/lib{LLVM,LTO}*.so* \
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
  depends=('llvm' "ocaml=$_ocaml_ver" 'ocaml-ctypes')

  cd "$srcdir/llvm-$pkgver.src"

  install -d "$pkgdir"/{usr/lib,usr/share/doc/$pkgname}
  cp -a "$srcdir/ocaml.lib" "$pkgdir/usr/lib/ocaml"
  cp -a "$srcdir/ocaml.doc" "$pkgdir/usr/share/doc/$pkgname/html"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lld() {
  pkgdesc="Linker from the LLVM project"
  url="http://lld.llvm.org/"
  depends=('llvm-libs')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/lld DESTDIR="$pkgdir" install

  # Remove documentation sources
  rm -r "$pkgdir"/usr/share/doc/$pkgname/html/{_sources,.buildinfo}

  install -Dm644 tools/$pkgname/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_lldb() {
  pkgdesc="Next generation, high-performance debugger"
  url="http://lldb.llvm.org/"
  depends=('llvm-libs' 'libxml2' 'python2' 'python2-six')

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

  install -Dm644 tools/$pkgname/LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  depends=('llvm-libs' 'gcc' 'libxml2')
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

  install -Dm644 tools/$pkgname/LICENSE.TXT \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_clang-tools-extra() {
  pkgdesc="Extra tools built using clang's tooling APIs"
  url="http://clang.llvm.org/"
  depends=('clang')

  cd "$srcdir/llvm-$pkgver.src"

  make -C build/tools/clang/tools/extra DESTDIR="$pkgdir" install

  # Remove documentation sources
  rm -r "$pkgdir"/usr/share/doc/clang-tools/html/{_sources,.buildinfo}

  # Use Python 2
  sed -i 's|/usr/bin/env python|&2|' \
    "$pkgdir"/usr/share/clang/{clang-tidy-diff,run-clang-tidy,run-find-all-symbols}.py

  install -Dm644 tools/clang/tools/extra/LICENSE.TXT \
    "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

# vim:set ts=2 sw=2 et:
