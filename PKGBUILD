## to run the project inside the container:
#  pacur build <target>

targets=(
  "centos"
  "ubuntu"
)
pkgname="envoyproxy"
pkgver="1.17.0"
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
  "https://dl.bintray.com/tetrate/getenvoy-deb/pool/stable/g/getenvoy-envoy/getenvoy-envoy_1.17.0.p0.g5c801b2-1p72.g28ef262_amd64.deb"
)
hashsums=(
  "9a4d25345c1e96f84d5e5a112d6aaa099b13a1ac514f7e33e085c757509e9c0036bc8305c51fa2db83744940a2ab76344d5098215bfeba197db0c63b57db5bb6"
)

package() {
  cd "${srcdir}"
  ar x getenvoy-envoy_*.deb
  tar -xvf "${srcdir}/data.tar.xz" -C "${pkgdir}"
  tar -xvf "${srcdir}/control.tar.gz" -C "${pkgdir}"
  rm -r "${pkgdir}/usr/share"
  rm -r "${pkgdir}/control"
}
