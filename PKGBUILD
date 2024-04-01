# Maintainer: pheiduck <pheiduck@github.com>

# Arch Linux Credits:
# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Contributor: Angel Velasquez <angvp@archlinux.org>
# Contributor: Eric Belanger <eric@archlinux.org>
# Contributor: Lucien Immink <l.immink@student.fnt.hvu.nl>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgbase=curl
pkgname=(curl libcurl-compat libcurl-gnutls)
pkgver=$(curl -s https://raw.githubusercontent.com/curl/curl/master/RELEASE-NOTES | sed -n "1p" | cut -c18-22)
pkgrel=$(date '+%Y%m%d')
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
             'patchelf'
             'glibc')

checkdepends=('valgrind')
provides=('libcurl.so')
validpgpkeys=('27EDEAF22F3ABCEB50DB9A125CC908FDB71E12C2') # Daniel Stenberg
source=("${url}/snapshots/${pkgbase}-${pkgver}-${pkgrel}.tar.xz")
sha512sums=('SKIP')

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

  "${srcdir}/${pkgbase}-${pkgver}-${pkgrel}"/configure \
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

  "${srcdir}/${pkgbase}-${pkgver}-${pkgrel}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols \
    --with-openssl \
    --with-openssl-quic
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc} -C lib
  patchelf --set-soname 'libcurl-compat.so.4' ./lib/.libs/libcurl.so

  # build libcurl-gnutls
  cd "${srcdir}"/build-curl-gnutls

  "${srcdir}/${pkgbase}-${pkgver}-${pkgrel}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols \
    --with-gnutls \
    --without-openssl
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -j${nproc} -C lib
  patchelf --set-soname 'libcurl-gnutls.so.4' ./lib/.libs/libcurl.so
}

check() {
  cd build-curl
  # -v: verbose
  # -a: keep going on failure (so we see everything which breaks, not just the first failing test)
  # -k: keep test files after completion
  # -am: automake style TAP output
  # -p: print logs if test fails
  # -j: parallelization
  # disable test 433, since it requires the glibc debug info
  make TFLAGS="-v -a -k -p -j$(nproc) !357" test-nonflaky
}

package_curl() {
  depends+=('openssl' 'libcrypto.so' 'libssl.so')
  provides=('libcurl.so')

  cd build-curl

  make -j${nproc} DESTDIR="${pkgdir}" install
  make -j${nproc} DESTDIR="${pkgdir}" install -C scripts

    cd "${srcdir}/${pkgname}-${pkgver}-${pkgrel}"

  # license
  install -Dt "${pkgdir}/usr/share/licenses/$pkgname" -m0644 COPYING
}

package_libcurl-compat() {
  pkgdesc='command line tool and library for transferring data with URLs (no versioned symbols)'
  depends=('curl')
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
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-compat
}

package_libcurl-gnutls() {
  pkgdesc='command line tool and library for transferring data with URLs (no versioned symbols, linked against gnutls)'
  depends=('curl' 'gnutls')
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
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-gnutls
}
