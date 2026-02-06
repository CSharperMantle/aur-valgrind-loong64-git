# Maintainer: csmantle <aur at csmantle dot top>
# Contributor: Fabio 'Lolix' Loli <lolix@disroot.org>
# Contributor: Ossi Saukko <osaukko at gmail dot com>

_name=valgrind
_reponame=valgrind-loongarch64
pkgname=valgrind-loong64-git
pkgver=3.26.0.r305.g7934a9bff
pkgrel=1
pkgdesc='A tool to help find memory-management problems in programs'
arch=('loong64')
url='http://valgrind.org/'
license=('GPL-2.0-or-later')
depends=('glibc' 'perl')
makedepends=('gdb' 'openmpi' 'git' 'docbook-xml' 'docbook-xsl' 'docbook-sgml')
optdepends=(
  'python: cg_* scripts'
)
options=('!emptydirs' '!strip')
provides=("$_name")
conflicts=("$_name")
source=('git+https://github.com/CSharperMantle/valgrind-loongarch64.git'
        '0001-Make-valgrind-respect-C-CPP-LD-FLAGS.patch')
b2sums=('SKIP'
        '06ad2aba854abc366c2e5a5b15c332b3f0cd9f998422a9c38c115d0a7e9c4d50f6720427ef97dbf549b3ea9cc5f344980594bcf842704c94d0223a998f9297a3')
options=(!lto) # https://bugs.kde.org/show_bug.cgi?id=338252

pkgver() {
  cd "$_reponame"
  #git describe --long --tags | sed -e 's|-|.|g' -e 's|VALGRIND_||g' -e 's|_|.|g'
  git describe --long --tags | sed 's/^VALGRIND_//;s/\([^-]*-g\)/r\1/;s/-/./g;s|_|.|g'
}

prepare() {
  cd "$_reponame"

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "$srcdir"/"$src"
  done

  sed -i 's|sgml/docbook/xsl-stylesheets|xml/docbook/xsl-stylesheets-1.79.2-nons|' docs/Makefile.am

  autoreconf -ifv
}

build() {
  cd "$_reponame"
  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --libexecdir=/usr/lib \
    --mandir=/usr/share/man \
    --enable-lto=yes
  make
  make -C docs man-pages
}

check() {
  # only run if glibc-debug is supplied manually
  if ! pacman -Q glibc-debug; then echo -e "\033[1;31mcheck() not run, supply glibc-debug if unintended!\033[0m"; return 0; fi

  cd "$_reponame"

  # Make sure a basic binary runs. There should be no errors.
  ./vg-in-place --error-exitcode=1 /bin/true

  # Make sure no extra FLAGS leak through, the testsuite
  # sets all flags necessary. See also configure above.
  make check CPPFLAGS= CFLAGS= CXXFLAGS= LDFLAGS=

  # XXX: run full regtest but only report issues some tests fail duo
  # current toolchain and expectations, take a manual look if its fine
  #echo "===============TESTING==================="
  #make regtest || true

  # Make sure test failures show up in build.log
  # Gather up the diffs (at most the first 20 lines for each one)
  #local f max_lines=20 diff_files=()
  #mapfile -d '' diff_files < <(find . -name '*.diff' -print0 | sort -z)
  #if (( ${#diff_files[@]} == 0 )); then
    #echo "Congratulations, all tests passed!"
  #else
    #warning "Some tests failed!"
    #for f in "${diff_files[@]}"; do
        #echo "================================================="
        #echo "${f}"
        #echo "================================================="
        #if (( $(wc -l < "${f}") < ${max_lines} )); then
          #cat "${f}"
        #else
          #head -n ${max_lines} "${f}"
          #echo "<truncated beyond ${max_lines} lines>"
        #fi
    #done | tee diffs
  #fi
  #echo "===============END TESTING==============="
}

package() {
  cd "$_reponame"
  make DESTDIR="$pkgdir" install

  install -Dm644 docs/*.1 -t "$pkgdir"/usr/share/man/man1
    if check_option 'debug' n; then
    find "$pkgdir"/usr/bin -type f -executable -exec strip "$STRIP_BINARIES" {} + || :
  fi
}

# vim: ts=2 sw=2 et:
