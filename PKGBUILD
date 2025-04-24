# Maintainer: pheiduck <pheiduck@github.com>

# Arch Linux Credits:
# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Contributor: Angel Velasquez <angvp@archlinux.org>
# Contributor: Eric Belanger <eric@archlinux.org>
# Contributor: Lucien Immink <l.immink@student.fnt.hvu.nl>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgbase=curl
pkgname=(curl-daily libcurl-compat-daily libcurl-gnutls-daily)
pkgver=$(curl -s https://raw.githubusercontent.com/curl/curl/master/include/curl/curlver.h | awk -F\" '/#define LIBCURL_VERSION/ {print $2}' | sed 's/-DEV//')
pkgrel=1
epoch=1
pkgdesc='command line tool and library for transferring data with URLs'
arch=('any')
url='https://curl.se'
license=('MIT')
depends=('ca-certificates'
         'brotli' 'libbrotlidec.so'
         'krb5' 'libgssapi_krb5.so'
         'libidn2' 'libidn2.so'
         'libnghttp2' 'libnghttp2.so'
         'libnghttp3' 'libnghttp3.so'
         'libpsl' 'libpsl.so'
         'libssh2' 'libssh2.so'
         'zlib' 'libz.so'
         'zstd' 'libzstd.so')
makedepends=('git' 
             'patchelf')
provides=('libcurl.so')
conflicts=('curl' 'libcurl-compat' 'libcurl-gnutls')
validpgpkeys=('27EDEAF22F3ABCEB50DB9A125CC908FDB71E12C2') # Daniel Stenberg
date=$(date '+%Y%m%d')
source=("${url}/snapshots/${pkgbase}-${pkgver}-${date}.tar.xz")
sha512sums=('SKIP')

# Workaround: rename uint_hash API to compiler-suggested uint_tbl API
prepare() {
  cd "${srcdir}/${pkgbase}-${pkgver}-${date}"
  sed -i \
    -e 's/struct uint_hash/struct uint_tbl/g' \
    -e 's/Curl_uint_hash_init/Curl_uint_tbl_init/g' \
    -e 's/Curl_uint_hash_destroy/Curl_uint_tbl_destroy/g' \
    -e 's/Curl_uint_hash_get/Curl_uint_tbl_get/g' \
    -e 's/Curl_uint_hash_set/Curl_uint_tbl_set/g' \
    -e 's/Curl_uint_hash_remove/Curl_uint_tbl_remove/g' \
    -e 's/Curl_uint_hash_visit/Curl_uint_tbl_visit/g' \
    -e 's/Curl_uint_hash_count/Curl_uint_tbl_count/g' \
    lib/vquic/curl_osslq.c
}

build() {
  local _configure_options=(
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

  mkdir -p build-curl{,-compat,-gnutls}

  # build curl
  cd "${srcdir}"/build-curl

  "${srcdir}/${pkgbase}-${pkgver}-${date}"/configure \
  "${_configure_options[@]}" \
    --enable-versioned-symbols \
    --with-fish-functions-dir=/usr/share/fish/vendor_completions.d/ \
    --with-openssl \
    --with-openssl-quic \
    --with-zsh-functions-dir=/usr/share/zsh/site-functions/
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc}
  # build libcurl-compat
  cd "${srcdir}"/build-curl-compat

  "${srcdir}/${pkgbase}-${pkgver}-${date}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols \
    --with-openssl \
    --with-openssl-quic
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc} -C lib
  patchelf --set-soname 'libcurl-compat.so.4' ./lib/.libs/libcurl.so

  # build libcurl-gnutls
  cd "${srcdir}"/build-curl-gnutls

  "${srcdir}/${pkgbase}-${pkgver}-${date}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols \
    --with-gnutls \
    --without-openssl
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc} -C lib
  patchelf --set-soname 'libcurl-gnutls.so.4' ./lib/.libs/libcurl.so
}

package_curl-daily() {
  depends+=('openssl' 'libcrypto.so' 'libssl.so')
  provides=('libcurl.so')

  cd "${srcdir}"/build-curl

  make -j${nproc} DESTDIR="${pkgdir}" install
  make -j${nproc} DESTDIR="${pkgdir}" install -C scripts

  cd "${srcdir}/${pkgbase}-${pkgver}-${date}"

  # license
  install -Dt "${pkgdir}/usr/share/licenses/$pkgname" -m0644 COPYING
}

package_libcurl-compat-daily() {
  pkgdesc='command line tool and library for transferring data with URLs (no versioned symbols)'
  depends=('curl-daily')
  provides=('libcurl-compat.so')

  cd "${srcdir}"/build-curl-compat

  make -j${nproc} -C lib DESTDIR="${pkgdir}" install

  mv "${pkgdir}"/usr/lib/libcurl{,-compat}.so.4.8.0
  rm "${pkgdir}"/usr/lib/libcurl.{a,so}*
  for version in 3 4.0.0 4.1.0 4.2.0 4.3.0 4.4.0 4.5.0 4.6.0 4.7.0; do
    ln -s libcurl-compat.so.4.8.0 "${pkgdir}"/usr/lib/libcurl.so.${version}
    ln -s libcurl-compat.so.4.8.0 "${pkgdir}"/usr/lib/libcurl-compat.so.${version}
  done

  install -dm 0755 "${pkgdir}"/usr/share/licenses
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-compat-daily
}

package_libcurl-gnutls-daily() {
  pkgdesc='command line tool and library for transferring data with URLs (no versioned symbols, linked against gnutls)'
  depends=('curl-daily' 'gnutls')
  provides=('libcurl-gnutls.so')

  cd "${srcdir}"/build-curl-gnutls

  make -j${nproc} -C lib DESTDIR="${pkgdir}" install

  mv "${pkgdir}"/usr/lib/libcurl{,-gnutls}.so.4.8.0
  rm "${pkgdir}"/usr/lib/libcurl.{a,so}*
  ln -s libcurl-gnutls.so.4 "${pkgdir}"/usr/lib/libcurl-gnutls.so
  for version in 3 4 4.0.0 4.1.0 4.2.0 4.3.0 4.4.0 4.5.0 4.6.0 4.7.0; do
    ln -s libcurl-gnutls.so.4.8.0 "${pkgdir}"/usr/lib/libcurl-gnutls.so.${version}
  done

  install -dm 0755 "${pkgdir}"/usr/share/licenses
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-gnutls-daily
}
