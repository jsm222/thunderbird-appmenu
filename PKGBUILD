# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Contributor: Ionut Biru <ibiru@archlinux.org>
# Contributor: Alexander Baldeck <alexander@archlinux.org>
# Contributor: Dale Blount <dale@archlinux.org>
# Contributor: Anders Bostrom <anders.bostrom@home.se>
# Additional patching: Nikita Tarasov <nikatar@disroot.org>


_pkgname=thunderbird
pkgname=thunderbird-appmenu
pkgver=60.8.0
pkgrel=1
pkgdesc="Thunderbird from extra with appmenu patch"
arch=(x86_64)
license=(MPL GPL LGPL)
url="https://aur.archlinux.org/packages/thunderbird-appmenu/"
depends=(gtk3 mozilla-common libxt startup-notification mime-types dbus-glib alsa-lib
         nss hunspell sqlite ttf-font icu)
makedepends=(unzip zip diffutils python2 yasm mesa imake libpulse inetutils xorg-server-xvfb
             autoconf2.13 rust clang llvm gtk2)
optdepends=('libcanberra: sound support')
provides=("thunderbird=$pkgver")
conflicts=("thunderbird")
options=(!emptydirs !makeflags)
source=(https://ftp.mozilla.org/pub/mozilla.org/thunderbird/releases/$pkgver/source/thunderbird-$pkgver.source.tar.xz{,.asc}
        rust-1.35.patch
        $_pkgname.desktop
        unity-menubar.patch)
sha256sums=('1e7a13e64b63476d2235aaac6823fdab949af45cfcd5a25ee710cbae08c2f5d1'
            'SKIP'
            '3257987cc0ab0559c65561d523eb4d0e73b5edb86d9d18a2487818d9fca0bb30'
            '3534ea85d8e0e35dba5f40a7a07844df19f3a480e1358fc50c2502f122dab789'
            '22786f52773b93046fdb12378f67343b1d7d3390e83a00ff4aa03948c2b1c9e2')
validpgpkeys=(14F26682D0916CDD81E37B6D61B7B526D98F0353) # Mozilla Software Releases <release@mozilla.com>

# Google API keys (see http://www.chromium.org/developers/how-tos/api-keys)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact foutrelis@archlinux.org for
# more information.
_google_api_key=AIzaSyDwr302FpOSkGRpLlUpPThNTDPbXcIn_FM

# Mozilla API keys (see https://location.services.mozilla.com/api)
# Note: These are for Arch Linux use ONLY. For your own distribution, please
# get your own set of keys. Feel free to contact heftig@archlinux.org for
# more information.
_mozilla_api_key=16674381-f021-49de-8622-3021c5942aff

prepare() {
  cd $_pkgname-$pkgver

  # # Fix build with rust 1.35 (Fedora)
  patch -Np1 -i ../rust-1.35.patch

  # actual appmenu patch from ubuntu repos
  patch -Np1 -i ../unity-menubar.patch

  echo -n "$_google_api_key" >google-api-key
  echo -n "$_mozilla_api_key" >mozilla-api-key

  cat >.mozconfig <<END
ac_add_options --enable-application=comm/mail
ac_add_options --enable-calendar

ac_add_options --prefix=/usr
ac_add_options --enable-release
ac_add_options --enable-linker=gold
ac_add_options --enable-hardening
ac_add_options --enable-optimize
# https://bugzilla.mozilla.org/show_bug.cgi?id=1521249
#ac_add_options --enable-rust-simd
# https://bugzilla.mozilla.org/show_bug.cgi?id=1423822
ac_add_options --disable-elf-hack

# Branding
ac_add_options --enable-official-branding
ac_add_options --enable-update-channel=release
ac_add_options --with-distribution-id=org.archlinux

# Keys
ac_add_options --with-google-location-service-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-google-safebrowsing-api-keyfile=${PWD@Q}/google-api-key
ac_add_options --with-mozilla-api-keyfile=${PWD@Q}/mozilla-api-key

# System libraries
ac_add_options --with-system-zlib
ac_add_options --with-system-bz2
ac_add_options --with-system-icu
ac_add_options --with-system-jpeg
ac_add_options --with-system-nspr
ac_add_options --with-system-nss
ac_add_options --enable-system-hunspell
ac_add_options --enable-system-sqlite
ac_add_options --enable-system-ffi

# Features
ac_add_options --enable-alsa
ac_add_options --disable-jack
ac_add_options --enable-startup-notification
ac_add_options --disable-crashreporter
ac_add_options --disable-updater
ac_add_options --disable-gconf
END

}

build() {
  cd $_pkgname-$pkgver
  ./mach configure
  ./mach build
  ./mach buildsymbols
}

package() {
  cd $_pkgname-$pkgver
  DESTDIR="$pkgdir" ./mach install

  _vendorjs="$pkgdir/usr/lib/$_pkgname/defaults/preferences/vendor.js"
  install -Dm644 /dev/stdin "$_vendorjs" <<END
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");

// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/hunspell");

// Disable default mailer checking.
pref("mail.shell.checkDefaultMail", false);

// Don't disable our bundled extensions in the application directory
pref("extensions.autoDisableScopes", 11);
pref("extensions.shownSelectionUI", true);
END

  _distini="$pkgdir/usr/lib/$_pkgname/distribution/distribution.ini"
  install -Dm644 /dev/stdin "$_distini" <<END
[Global]
id=archlinux
version=1.0
about=Mozilla Thunderbird for Arch Linux

[Preferences]
app.distributor=archlinux
app.distributor.channel=$_pkgname
app.partner.archlinux=archlinux
END

  for i in 16 22 24 32 48 64 128 256; do
    install -Dm644 comm/mail/branding/thunderbird/default${i}.png \
      "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/$_pkgname.png"
  done
  install -Dm644 comm/mail/branding/thunderbird/TB-symbolic.svg \
    "$pkgdir/usr/share/icons/hicolor/symbolic/apps/thunderbird-symbolic.svg"

  install -Dm644 ../$_pkgname.desktop \
    "$pkgdir/usr/share/applications/$_pkgname.desktop"

  # Use system-provided dictionaries
  rm -r "$pkgdir/usr/lib/$_pkgname/dictionaries"
  ln -Ts /usr/share/hunspell "$pkgdir/usr/lib/$_pkgname/dictionaries"
  ln -Ts /usr/share/hyphen "$pkgdir/usr/lib/$_pkgname/hyphenation"

  # Install a wrapper to avoid confusion about binary path
  install -Dm755 /dev/stdin "$pkgdir/usr/bin/$_pkgname" <<END
#!/bin/sh
exec /usr/lib/$_pkgname/thunderbird "\$@"
END

  # Replace duplicate binary with wrapper
  # https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -srf "$pkgdir/usr/bin/$_pkgname" \
    "$pkgdir/usr/lib/$_pkgname/thunderbird-bin"
}
