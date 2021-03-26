## to run the project inside the container:
#  pacur build <target>

targets=(
  "centos"
  "ubuntu"
)
pkgname="envoyproxy"
pkgver="1.16.2"
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
  "https://dl.bintray.com/tetrate/getenvoy-deb/pool/stable/g/getenvoy-envoy/getenvoy-envoy_1.16.2.p0.ge98e41a-1p71.gbe6132a_amd64.deb"
)
hashsums=(
  "daa48db48801aba772aaf61b09de943019b00347fc7f905889398f4688c5d84975e6f83cb39114f84307c67dabfadc31c8f71f00aaf76eba5529bcfbcf58d2bc"
)

package() {
  cd "${srcdir}"
  ar x getenvoy-envoy_*.deb
  tar -xvf "${srcdir}/data.tar.xz" -C "${pkgdir}"
  tar -xvf "${srcdir}/control.tar.gz" -C "${pkgdir}"
  rm -r "${pkgdir}/usr/share"
  rm -r "${pkgdir}/control"
}
