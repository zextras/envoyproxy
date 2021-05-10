## to run the project inside the container:
#  pacur build <target>

targets=(
  "centos"
  "ubuntu"
)
pkgname="envoyproxy"
pkgver="1.18.2"
pkgrel="1"
pkgdesc="A high performance, open source, general RPC framework that puts mobile and HTTP/2 first."
pkgdesclong=(
  "A high performance, open source, general RPC framework that puts mobile and HTTP/2 first."
)
maintainer="Zextras SRL <packages@zextras.com>"
arch="amd64"
url="https://www.zextras.com"
license=("Apache2")
section="utils"
provides=("getenvoy-envoy")
conflicts=("getenvoy-envoy")
sources=(
  "https://dl.bintray.com/tetrate/getenvoy-deb/pool/stable/g/getenvoy-envoy/getenvoy-envoy_1.18.2.p0.gd362e79-1p75.g76c310e_amd64.deb"
)
hashsums=(
  "8dcfbc4d74655a8de40c266e3d803ddb238028dc96ad7168d03512107ae46d58"
)

package() {
  cd "${srcdir}"
  ar x getenvoy-envoy_*.deb
  tar -xvf "${srcdir}/data.tar.xz" -C "${pkgdir}"
  tar -xvf "${srcdir}/control.tar.gz" -C "${pkgdir}"
  rm -r "${pkgdir}/usr/share"
  rm -r "${pkgdir}/control"
}
