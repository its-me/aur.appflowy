# Maintainer: Sergey Kanafyev <sergeykanafyev@gmail.com>

pkgname=appflowy
pkgver=0.12.1
pkgrel=1
pkgdesc="Open-source alternative to Notion – you own your data and customizations"
arch=('x86_64')
url="https://appflowy.com"
license=('AGPL-3.0-or-later')
conflicts=('appflowy-bin')
depends=(
    'glib2>=2.80'
    'gst-plugins-base-libs'
    'gtk3'
    'hicolor-icon-theme'
    'libkeybinder3'
    'libnotify'
    'rocksdb'
)
_flutter_ver=3.27.4
makedepends=(
    'clang'
    'cmake'
    'ninja'
    'pkg-config'
    'sqlite'
    'openssl'
    'unzip'
    'protobuf'
    'rustup'
    'cargo-make'
)
optdepends=(
    'kdialog: file picker on KDE Plasma'
    'zenity: file picker on GNOME/GTK'
)
options=('!lto' '!debug' '!buildflags')
source=(
    "${pkgname}-${pkgver}.tar.gz::https://github.com/AppFlowy-IO/AppFlowy/archive/refs/tags/${pkgver}.tar.gz"
    "flutter-${_flutter_ver}-linux.tar.xz::https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${_flutter_ver}-stable.tar.xz"
    "appflowy.desktop"
)
sha256sums=(
    '6c754ea233e51a1ef085975d3463eee449488fe07e22abc42900eee68073a640'
    '64df4273de625433c7ba41967932b782f5f9abf3199db8330782d64508379344'
    '55c02d13249b333088ee452e76c8f36254e510651023549dc7e35efca02ca821'
)

prepare() {
    export PATH="${srcdir}/flutter/bin:${PATH}"

    cd "AppFlowy-${pkgver}/frontend"

    # Install the Rust toolchain declared in rust-toolchain.toml (channel = "1.85")
    rustup toolchain install --no-self-update
    rustup target add x86_64-unknown-linux-gnu

    # Pre-fetch Rust crate dependencies
    cargo fetch --manifest-path=rust-lib/Cargo.toml

    # Fetch Flutter package dependencies
    cd appflowy_flutter
    flutter pub get

    # Run code generation explicitly so it runs visibly and before cargo-make
    cd ..
    ./scripts/code_generation/generate.sh --skip-pub-get
}

build() {
    export PATH="${srcdir}/flutter/bin:${HOME}/.pub-cache/bin:${PATH}"
    export CC=clang
    export CXX=clang++
    export ROCKSDB_LIB_DIR=/usr/lib

    cd "AppFlowy-${pkgver}/frontend"
    cargo make --profile production-linux-x86_64 appflowy
}

package() {
    cd "AppFlowy-${pkgver}/frontend"

    # APPFLOWY_VERSION in Makefile.toml determines the product subdirectory name
    local _appver
    _appver=$(sed -n 's/^APPFLOWY_VERSION = "\(.*\)"/\1/p' Makefile.toml)
    local _product="appflowy_flutter/product/${_appver}/linux/Release/AppFlowy"

    # Install AppFlowy bundle
    install -dm755 "${pkgdir}/usr/lib/AppFlowy"
    cp -r "${_product}/." "${pkgdir}/usr/lib/AppFlowy/"
    chmod 755 "${pkgdir}/usr/lib/AppFlowy/AppFlowy"

    # Symlink into PATH
    install -dm755 "${pkgdir}/usr/bin"
    ln -s "/usr/lib/AppFlowy/AppFlowy" "${pkgdir}/usr/bin/appflowy"

    # Desktop entry
    install -Dm644 "${srcdir}/appflowy.desktop" \
        "${pkgdir}/usr/share/applications/appflowy.desktop"

    # Icons
    install -Dm644 "appflowy_flutter/linux/packaging/assets/logo.png" \
        "${pkgdir}/usr/share/icons/hicolor/256x256/apps/appflowy.png"
    install -Dm644 "appflowy_flutter/assets/images/flowy_logo.svg" \
        "${pkgdir}/usr/share/icons/hicolor/scalable/apps/appflowy.svg"

    install -Dm644 "../LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
