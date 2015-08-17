# Maintainer: Auguste Pop <auguste [at] gmail [dot] com>

pkgname=ofetion
_PKGALIAS=openfetion
pkgver=2.2.2
_libver=2.2.2
_cliver=2.2.0
_guiver=2.2.1
pkgrel=1
pkgdesc="Fetion client for linux"
arch=('i686' 'x86_64')
url="http://basiccoder.com/openfetion"
license=('GPL')
depends=('sqlite3' 'gtk2')
makedepends=('cmake')
optdepends=('libnotify: notification support'
            'gstreamer0.10: multimedia support'
            'libxss: idle detection support'
            'networkmanager: auto-reconnect support')
conflicts=('openfetion' 'openfetion-all' 'openfetion-cli'
           'openfetion-standalone')
install=$pkgname.install
_libsrc="lib$pkgname-$_libver"
_clisrc="cli$pkgname-$_cliver"
_guisrc="$_PKGALIAS-$_guiver"
source=("http://$pkgname.googlecode.com/files/$_libsrc.tar.gz"
        "http://$pkgname.googlecode.com/files/$_clisrc.tar.gz"
        "http://$pkgname.googlecode.com/files/$_guisrc.tar.gz")
md5sums=('1939979abbad1618ffbb64078badb32e'
         '1fc2e19da2e1facd4086c6a6b8b81eb5'
         'a5d4fcb2a8a1cdf39cca7a116b596558')

_ods=()
_tmpfile()
{
    echo "$srcdir/.tmp"
}

_libbuild()
{
    echo "$srcdir/lib$pkgname-build"
}

_clibuild()
{
    echo "$srcdir/cli$pkgname-build"
}

_guibuild()
{
    echo "$srcdir/gui$pkgname-build"
}

add_ods()
{
    pkg-config $1 && echo "$3" >> "$(_tmpfile)" || _ods=("${_ods[@]}" "$2")
}

clear_enter()
{
    rm -rf "$1"
    mkdir "$1"
    cd "$1"
}

force_dependency()
{
    sed -i "/pkg_check_modules(OFETION /{
        a\
            set(OFETION_LIBRARIES $(_libbuild)/libofetion.so \${OFETION_LIBRARIES})
        c\
            pkg_check_modules(OFETION REQUIRED $2)
    }" "$1/CMakeLists.txt"
}

unset_optdepend()
{
    for _idx in ${!optdepends[@]}
    do
        if [[ "${optdepends[$_idx]}" =~ ^$1:* ]]
        then
            unset optdepends[$_idx]
            break
        fi
    done
}

build()
{
    unset CFLAGS

    clear_enter "$(_libbuild)"
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release \
        "$srcdir/$_libsrc"
    make

    _libdepends="$(sed -n 's/^Requires: \(.*\)$/\1/p' $(_libbuild)/ofetion.pc)"

    clear_enter "$(_clibuild)"
    force_dependency "$srcdir/$_clisrc" "$_libdepends"
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release \
        "$srcdir/$_clisrc"
    make CPATH="$srcdir/$_libsrc"

    clear_enter "$(_guibuild)"
    rm -rf "$(_tmpfile)"
    add_ods libnotify "-DWITH_LIBNOTIFY=OFF" libnotify
    add_ods gstreamer-0.10 "-DWITH_GSTREAMER=OFF" gstreamer0.10
    add_ods xscrnsaver "-DWITH_LIBXSS=OFF" libxss
    add_ods NetworkManager "-DWITH_NETWORKMANAGER=OFF" networkmanager
    force_dependency "$srcdir/$_guisrc" "$_libdepends"
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release \
        "${_ods[@]}" -DWITH_INDICATE=OFF "$srcdir/$_guisrc"
    make CPATH="$srcdir/$_libsrc"
}

package()
{
    cd "$(_libbuild)"
    make DESTDIR="$pkgdir" install

    cd "$(_clibuild)"
    make DESTDIR="$pkgdir" install

    cd "$(_guibuild)"
    make DESTDIR="$pkgdir" install

    while read _ch
    do
        if [[ 1 -gt 0 ]]; then depends=("${depends[@]}" "$_ch"); fi
        unset_optdepend "$_ch"
    done < "$(_tmpfile)"
    optdepends=("${optdepends[@]}")
    rm -rf "$(_tmpfile)"
}
