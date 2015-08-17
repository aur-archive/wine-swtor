# Based on the Wine package in the community repository.
# Contributor: fettouhi <A.Fettouhi *AT* gmail *DOT* com>
# Maintainer: TheBenj <thebenj88 *AT* gmail *DOT* com>
pkgname=wine-swtor
pkgver=1.5.27
pkgrel=1

_pkgbasever=${pkgver/rc/-rc}

source=(http://prdownloads.sourceforge.net/wine/wine-$pkgver.tar.bz2
		proof_of_concept.patch
		wine.install)
sha256sums=('ca2bd31d15b6ddb7f900c813ed113b732a61057d7e1669ab37494284c259d4b8'
            'bc767d0199c5cf7b48c1c49a3e5574348113cb883a1e60602f5475ef223f3b8f'
            '802fe2ec9098ddbf3e5736ca8d8ad80b0d771953e39c55da9e835fe9942364f8')

pkgdesc="Wine with patch for logging in Star Wars: The Old Republic"
url="http://www.winehq.com"
arch=(i686 x86_64)
license=(LGPL)
install=wine.install

depends=(
  fontconfig      lib32-fontconfig
  mesa            lib32-mesa 
  libxcursor      lib32-libxcursor
  libxrandr       lib32-libxrandr
  libxdamage      lib32-libxdamage
  libxi           lib32-libxi
  gettext         lib32-gettext
  desktop-file-utils
)

makedepends=(autoconf ncurses bison perl fontforge flex prelink
  'gcc>=4.5.0-2'  'gcc-multilib>=4.5.0-2'
  giflib          lib32-giflib
  libpng          lib32-libpng
  gnutls          lib32-gnutls
  libxinerama     lib32-libxinerama
  libxcomposite   lib32-libxcomposite
  libxmu          lib32-libxmu
  libxxf86vm      lib32-libxxf86vm
  libxml2         lib32-libxml2
  libldap         lib32-libldap
  lcms            lib32-lcms
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  alsa-lib        lib32-alsa-lib
  oss
  samba
)
  
optdepends=(
  giflib          lib32-giflib
  libpng          lib32-libpng
  libldap         lib32-libldap
  gnutls          lib32-gnutls
  lcms            lib32-lcms
  libxml2         lib32-libxml2
  mpg123          lib32-mpg123
  openal          lib32-openal
  v4l-utils       lib32-v4l-utils
  libpulse        lib32-libpulse
  alsa-plugins    lib32-alsa-plugins
  alsa-lib        lib32-alsa-lib
  oss             cups
  samba
)

if [[ $CARCH == i686 ]]; then
  # Strip lib32 etc. on i686
  depends=(${depends[@]/*32-*/})
  makedepends=(${makedepends[@]/*32-*/})
  makedepends=(${makedepends[@]/*-multilib*/})
  optdepends=(${optdepends[@]/*32-*/})
else
  provides=("wine=$pkgver" "bin32-wine=$pkgver" "wine-wow64=$pkgver")
  conflicts=('wine' 'bin32-wine' 'wine-wow64')
  replaces=('wine' 'bin32-wine')
fi


build() {
  cd "$srcdir"

  # Allow ccache to work
  mv wine-$_pkgbasever $pkgname

  # Patching
  cp ../*.patch "$srcdir/$pkgname"
  cd "$srcdir/$pkgname"
  patch -p1 < proof_of_concept.patch
  cd "$srcdir"

# Get rid of old build dirs
  rm -rf $pkgname-{32,64}-build
  mkdir $pkgname-32-build

  # These additional CFLAGS solve FS#27662
  export CFLAGS="${CFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"
  export CXXFLAGS="${CFLAGS/-D_FORTIFY_SOURCE=2/} -D_FORTIFY_SOURCE=0"

  if [[ $CARCH == x86_64 ]]; then
    msg2 "Building Wine-64..."

    mkdir $pkgname-64-build
    cd "$srcdir/$pkgname-64-build"
    ../$pkgname/configure \
      --prefix=/usr \
      --sysconfdir=/etc \
      --libdir=/usr/lib \
      --with-x \
      --enable-win64

    make

    _wine32opts=(
      --libdir=/usr/lib32
      --with-wine64="$srcdir/$pkgname-64-build"
    )

    export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  fi

  msg2 "Building Wine-32..."
  cd "$srcdir/$pkgname-32-build"
  ../$pkgname/configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --with-x \
    "${_wine32opts[@]}"

  # These additional CFLAGS solve FS#27560
  make CFLAGS+="-mstackrealign" CXXFLAGS+="-mstackrealign"
}

package() {
  msg2 "Packaging Wine-32..."
  cd "$srcdir/$pkgname-32-build"

  if [[ $CARCH == i686 ]]; then
    make prefix="$pkgdir/usr" install
  else
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib32" \
      dlldir="$pkgdir/usr/lib32/wine" install

    msg2 "Packaging Wine-64..."
    cd "$srcdir/$pkgname-64-build"
    make prefix="$pkgdir/usr" \
      libdir="$pkgdir/usr/lib" \
      dlldir="$pkgdir/usr/lib/wine" install
  fi
}

# vim:set ts=8 sts=2 sw=2 et:
