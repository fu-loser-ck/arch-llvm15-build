# Maintainer: Evangelos Foutras <foutrelis@gmail.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Sebastian Nowicki <sebnow@gmail.com>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Kieslich <tobias@justdreams.de>
# Contributor: Geoffroy Carrier <geoffroy.carrier@aur.archlinux.org>
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Gerardo Exequiel Pozzi <vmlinuz386@yahoo.com.ar>

pkgname=('llvm' 'llvm-ocaml' 'clang' 'clang-analyzer')
pkgver=2.9
_gcc_ver=4.6.1
pkgrel=6
arch=('i686' 'x86_64')
url="http://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('gcc-libs' 'libffi' 'python2' 'ocaml' "gcc=$_gcc_ver")
source=(http://llvm.org/releases/$pkgver/$pkgname-$pkgver.tgz
        http://llvm.org/releases/$pkgver/clang-$pkgver.tgz
        ftp://ftp.archlinux.org/other/community/clang/gcc-headers-4.5.2.tar.xz
        clang-plugin-loader-registry.patch
        cindexer-clang-path.patch
        clang-toolchains-gcc-versions.patch
        clang-pure64.patch
        enable-lto.patch
        bug-9869-operator-h-c++0x.patch)
md5sums=('793138412d2af2c7c7f54615f8943771'
         '634de18d04b7a4ded19ec4c17d23cfca'
         '70e23a3dc2b38ecb2bb4d2c48f47295d'
         '02c23b4aaca3445b8bf39fddb2f9906e'
         '87a7162dbe99e9ffce6c40bd09f5f4f0'
         '016d70145d52255c9a46fedde51634e2'
         '225ee6b531f8327f34f344a18cb4ec81'
         '8f7582d7440e4a8342c3aea9ec714fb4'
         '047cac563a557463d7ec6bd87d953f5e')

build() {
  cd "$srcdir/$pkgname-$pkgver"

  # At the present, clang must reside inside the LLVM source code tree to build
  # See http://llvm.org/bugs/show_bug.cgi?id=4840
  rm -rf tools/clang
  cp -r "$srcdir/clang-$pkgver" tools/clang

  # Fix symbolic links from OCaml bindings to LLVM libraries
  sed -i 's:\$(PROJ_libdir):/usr/lib/llvm:' bindings/ocaml/Makefile.ocaml

  # Fix installation directories, ./configure doesn't seem to set them right
  sed -i -e 's:\$(PROJ_prefix)/etc/llvm:/etc/llvm:' \
         -e 's:\$(PROJ_prefix)/lib:$(PROJ_prefix)/lib/llvm:' \
         -e 's:\$(PROJ_prefix)/docs/llvm:$(PROJ_prefix)/share/doc/llvm:' \
    Makefile.config.in

  # Fix insecure rpath (http://bugs.archlinux.org/task/14017)
  sed -i 's:$(RPATH) -Wl,$(\(ToolDir\|LibDir\|ExmplDir\))::g' Makefile.rules

  # Get the correct list of symbols to export
  # See http://lists.cs.uiuc.edu/pipermail/cfe-dev/2010-April/008559.html
  patch -Np1 -i "$srcdir/clang-plugin-loader-registry.patch"

  # Fix clang path in CIndexer.cpp (https://bugs.archlinux.org/task/22799)
  patch -d tools/clang -Np0 -i "$srcdir/cindexer-clang-path.patch"

  # Add GCC 4.6.1 to GccVersions (FS#23631)
  patch -d tools/clang -Np1 -i "$srcdir/clang-toolchains-gcc-versions.patch"

  if [[ $CARCH == x86_64 ]]; then
    # Adjust lib paths
    patch -d tools/clang -Np0 -i "$srcdir/clang-pure64.patch"
  fi

  # Make -flto work
  # Use gold instead of default linker, and always use the plugin
  patch -d tools/clang -Np0 -i "$srcdir/enable-lto.patch"

  # Fix upstream bug #9869:
  #   Operator.h incompatibility with GCC 4.6 in C++0x mode
  patch -Np2 -i "$srcdir/bug-9869-operator-h-c++0x.patch"

  # Apply strip option to configure
  _optimized_switch="enable"
  [[ $(check_option strip) == n ]] && _optimized_switch="disable"

  # Include location of libffi headers in CPPFLAGS
  export CPPFLAGS="$CPPFLAGS $(pkg-config --cflags libffi)"

  # TODO: Uncomment when clang works with GCC 4.6+
  #_cxx_headers="/usr/include/c++/$_gcc_ver"
  #if [[ ! -d $_cxx_headers ]]; then
  #  error "Couldn't find the C++ headers, PKGBUILD needs fixing!"
  #  return 1
  #fi
  _cxx_headers="/usr/include/c++/clang-$pkgver"

  _32bit_headers=""
  if [[ $CARCH == x86_64 ]]; then
    # Important for multilib
    _32bit_headers="32"
  fi

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
    --with-cxx-include-root=$_cxx_headers \
    --with-cxx-include-arch=$CHOST \
    --with-cxx-include-32bit-dir=$_32bit_headers \
    --$_optimized_switch-optimized

  make REQUIRES_RTTI=1
}

package_llvm() {
  pkgdesc="Low Level Virtual Machine"
  depends=('perl' 'libffi')

  cd "$srcdir/$pkgname-$pkgver"

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

  # Fix libdir in llvm-config (http://bugs.archlinux.org/task/14487)
  sed -i 's:\(ABS_RUN_DIR/lib\):\1/llvm:' "$pkgdir/usr/bin/llvm-config"

  # Get rid of example Hello transformation
  rm "$pkgdir"/usr/lib/llvm/*LLVMHello.*

  # Symlink the gold plugin where clang expects it
  ln -s llvm/LLVMgold.so "$pkgdir/usr/lib/LLVMgold.so"

  # Add ld.so.conf.d entry
  install -d "$pkgdir/etc/ld.so.conf.d"
  echo /usr/lib/llvm >"$pkgdir/etc/ld.so.conf.d/llvm.conf"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
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

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/llvm-ocaml/LICENSE"
}

package_clang() {
  pkgdesc="C language family frontend for LLVM"
  url="http://clang.llvm.org/"
  # It looks like clang still needs GCC to assemble and link object files
  # See http://old.nabble.com/%22clang--v%22-shows-a-GCC-call-td28378453.html
  depends=("llvm=$pkgver-$pkgrel" "gcc=$_gcc_ver")

  # Fix installation path for clang docs
  sed -i 's:$(PROJ_prefix)/share/doc/llvm:$(PROJ_prefix)/share/doc/clang:' \
    "$srcdir/llvm-$pkgver/Makefile.config"

  cd "$srcdir/llvm-$pkgver/tools/clang"
  make DESTDIR="$pkgdir" install

  # Fix permissions of static libs
  chmod -x "$pkgdir"/usr/lib/llvm/*.a

  # Revert the path change in case we want to do a repackage later
  sed -i 's:$(PROJ_prefix)/share/doc/clang:$(PROJ_prefix)/share/doc/llvm:' \
    "$srcdir/llvm-$pkgver/Makefile.config"

  # Install old libstdc++ headers. Contains combined headers from
  # gcc 4.5.2-6-i686 and gcc-multilib-4.5.2-6-x86_64
  install -d "$pkgdir/usr/include/c++"
  cp -rd "$srcdir/gcc-headers-4.5.2" "$pkgdir/usr/include/c++/clang-$pkgver"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/clang/LICENSE"
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

  # Use Python 2
  sed -i 's/env python$/\02/' \
    "$pkgdir/usr/lib/clang-analyzer/scan-view/scan-view" \
    "$pkgdir/usr/lib/clang-analyzer/scan-build/set-xcode-analyzer"

  # Compile Python scripts
  python2 -m compileall "$pkgdir/usr/lib/clang-analyzer"
  python2 -O -m compileall "$pkgdir/usr/lib/clang-analyzer"

  install -Dm644 LICENSE.TXT "$pkgdir/usr/share/licenses/clang-analyzer/LICENSE"
}

# vim:set ts=2 sw=2 et:
