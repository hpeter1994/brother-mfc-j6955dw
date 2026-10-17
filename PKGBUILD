# Maintainer: hpeter94 <hpeter1994@gmail.com>
# Built from Brother's official mfcj6955dwpdrv-3.5.0-1.i386.deb (the "i386" deb
# also ships x86_64 binaries; only those are installed here).

pkgname=brother-mfc-j6955dw
_model=mfcj6955dw
pkgver=3.5.0
pkgrel=1
pkgdesc="Brother MFC-J6955DW CUPS printer driver (patched for Arch)"
arch=('x86_64')
url="https://support.brother.com/"
license=('LicenseRef-Brother' 'GPL-2.0-or-later')
depends=('cups' 'ghostscript' 'perl' 'file')
options=('!strip' '!debug')
install=${pkgname}.install
source=("${_model}pdrv-${pkgver}-1.i386.deb"
        "arch-fixes.patch")
noextract=("${_model}pdrv-${pkgver}-1.i386.deb")
sha256sums=('3c6fab89e26538d6273f28d74768d7e78be219adb474183c9319b63fa5eb24e2'
            'd0db7c67d001ed155676cee31e29581a2e6ef337cf604843d5b1a9eeb0a7bd6c')

prepare() {
  rm -rf data && mkdir data
  bsdtar -xf "${_model}pdrv-${pkgver}-1.i386.deb" data.tar.gz
  bsdtar -xf data.tar.gz -C data
  cd "data/opt/brother/Printers/${_model}"
  patch -p1 -i "${srcdir}/arch-fixes.patch"
}

package() {
  local _p=/opt/brother/Printers/${_model}

  cp -a "${srcdir}/data/opt" "${pkgdir}/"
  chown -R root:root "${pkgdir}/opt"
  # 32-bit binaries and the LPD/printcap + lpadmin(USB) install helpers are not used
  rm -rf "${pkgdir}${_p}/lpd/i686"
  rm -f "${pkgdir}${_p}/cupswrapper/cupswrapper${_model}" "${pkgdir}${_p}/inf/setupPrintcapij"

  # What Brother's postinst does at install time
  ln -s "${_p}/lpd/x86_64/br${_model}filter"      "${pkgdir}${_p}/lpd/br${_model}filter"
  ln -s "${_p}/lpd/x86_64/brprintconf_${_model}"  "${pkgdir}${_p}/lpd/brprintconf_${_model}"
  install -dm755 "${pkgdir}/usr/bin"
  ln -s "${_p}/lpd/brprintconf_${_model}" "${pkgdir}/usr/bin/brprintconf_${_model}"

  # What cupswrapper<model> does. The CUPS filter must be an absolute
  # symlink: the wrapper finds its data dir with `readlink $0`.
  install -dm755 "${pkgdir}/usr/lib/cups/filter"
  ln -s "${_p}/cupswrapper/brother_lpdwrapper_${_model}" \
        "${pkgdir}/usr/lib/cups/filter/brother_lpdwrapper_${_model}"
  install -Dm644 "${pkgdir}${_p}/cupswrapper/brother_${_model}_printer_en.ppd" \
                 "${pkgdir}/usr/share/ppd/Brother/brother_${_model}_printer_en.ppd"

  install -Dm644 "${pkgdir}${_p}/LICENSE_ENG.txt" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
