# Maintainer: pheiduck <pheiduck@github.com>

# Arch Linux Credits:
# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Contributor: Angel Velasquez <angvp@archlinux.org>
# Contributor: Eric Belanger <eric@archlinux.org>
# Contributor: Lucien Immink <l.immink@student.fnt.hvu.nl>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgbase=curl
pkgname=(curl libcurl-compat libcurl-gnutls)
pkgver=8.2.0
pkgrel=$(date '+%Y%m%d')
pkgdesc='An URL retrieval utility and library'
arch=('any')
url='https://curl.se'
license=('MIT')
depends=('ca-certificates' 'brotli' 'libbrotlidec.so' 'krb5' 'libgssapi_krb5.so'
         'libidn2' 'libidn2.so' 'libnghttp2' 'libpsl' 'libpsl.so' 'libssh2' 'libssh2.so'
         'openssl' 'zlib' 'zstd' 'libzstd.so')
makedepends=('patchelf')
provides=('libcurl.so')
source=("${url}/snapshots/${pkgbase}-${pkgver}-${pkgrel}.tar.xz")
sha512sums=('SKIP')
validpgpkeys=('27EDEAF22F3ABCEB50DB9A125CC908FDB71E12C2') # Daniel Stenberg

_configure_options=(
  --prefix='/usr'
  --mandir='/usr/share/man'
  --disable-ldap
  --disable-ldaps
  --disable-manual
  --enable-ipv6
  --enable-threaded-resolver
  --with-gssapi
  --with-libssh2
  --with-random='/dev/urandom'
  --with-ca-bundle='/etc/ssl/certs/ca-certificates.crt'
)

build() {
  mkdir build-curl{,-compat,-gnutls}

  # build curl
  cd "${srcdir}"/build-curl

  "${srcdir}/${pkgbase}-${pkgver}-${pkgrel}"/configure \
    "${_configure_options[@]}" \
    --with-openssl \
    --enable-versioned-symbols
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc}

  # build libcurl-compat
  cd "${srcdir}"/build-curl-compat

  "${srcdir}/${pkgbase}-${pkgver}-${pkgrel}"/configure \
    "${_configure_options[@]}" \
    --with-openssl \
    --disable-versioned-symbols
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc} -C lib

  # build libcurl-gnutls
  cd "${srcdir}"/build-curl-gnutls

  "${srcdir}/${pkgbase}-${pkgver}-${pkgrel}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols \
    --without-openssl \
    --with-gnutls
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc} -C lib
  patchelf --set-soname 'libcurl-gnutls.so.4' ./lib/.libs/libcurl.so
}

package_curl() {
  cd build-curl

  make -j${nproc} DESTDIR="${pkgdir}" install
  make -j${nproc} DESTDIR="${pkgdir}" install -C scripts

    cd "${srcdir}/${pkgname}-${pkgver}-${pkgrel}"

  # license
  install -Dt "${pkgdir}/usr/share/licenses/$pkgname" -m0644 COPYING
}

package_libcurl-compat() {
  pkgdesc='An URL retrieval library (without versioned symbols)'
  depends=('curl' 'openssl')

  cd "${srcdir}"/build-curl-compat

  make -j${nproc} -C lib DESTDIR="${pkgdir}" install

  mv "${pkgdir}"/usr/lib/libcurl{,-compat}.so.4.8.0
  rm "${pkgdir}"/usr/lib/libcurl.{a,so}*
  for version in 3 4.0.0 4.1.0 4.2.0 4.3.0 4.4.0 4.5.0 4.6.0 4.7.0; do
    ln -s libcurl-compat.so.4.8.0 "${pkgdir}"/usr/lib/libcurl.so.${version}
  done

  install -dm 0755 "${pkgdir}"/usr/share/licenses
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-compat
}

package_libcurl-gnutls() {
  pkgdesc='An URL retrieval library (without versioned symbols and linked against gnutls)'
  depends=('curl' 'gnutls')

  cd "${srcdir}"/build-curl-gnutls

  make -j${nproc} -C lib DESTDIR="${pkgdir}" install

  mv "${pkgdir}"/usr/lib/libcurl{,-gnutls}.so.4.8.0
  rm "${pkgdir}"/usr/lib/libcurl.{a,so}*
  ln -s libcurl-gnutls.so.4 "${pkgdir}"/usr/lib/libcurl-gnutls.so
  for version in 3 4 4.0.0 4.1.0 4.2.0 4.3.0 4.4.0 4.5.0 4.6.0 4.7.0; do
    ln -s libcurl-gnutls.so.4.8.0 "${pkgdir}"/usr/lib/libcurl-gnutls.so.${version}
  done

  install -dm 0755 "${pkgdir}"/usr/share/licenses
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-gnutls
}
