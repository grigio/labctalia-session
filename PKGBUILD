# Maintainer: grigio <grigio.org>
#
# Package for Labctalia: a minimal labwc + Noctalia Wayland desktop. It
# installs the launcher, the XDG session entry, the systemd user units, the
# default labwc configuration/theme and the optional greeter helper from
# files/ in this repo. Nothing is downloaded — the repo is the source tree, so
# $startdir is used in package() and there is no source array.
#
# The package does NOT edit /etc/greetd and does NOT make Labctalia the default
# session; `sudo labctalia-session-default` does that explicitly.
#
#   makepkg -f       # -> labctalia-session-<pkgver>-<pkgrel>-any.pkg.tar.zst
#   makepkg -fi      # ... and install it
#
# pkgver comes from the nearest v* tag and pkgrel from the number of commits
# that touched packaged files, so a local build matches the CI-built package.
# Both can be overridden, as .github/workflows/arch-package.yml does:
#
#   SESSION_VERSION=1.0.0 SESSION_PKGREL=1 makepkg -f

pkgname=labctalia-session
pkgver=${SESSION_VERSION:-$(git -C "${startdir:-$PWD}" describe --tags --abbrev=0 2>/dev/null | sed 's/^v//')}
pkgver=${pkgver:-1.0.0}
_pkgrel=$(git -C "${startdir:-$PWD}" rev-list --count HEAD -- \
  PKGBUILD labctalia-session.install files README.md LICENSE \
  scripts/set-default-greetd.sh scripts/install.sh 2>/dev/null || true)
pkgrel=${SESSION_PKGREL:-${_pkgrel:-1}}
(( pkgrel >= 1 )) || pkgrel=1

pkgdesc='Labctalia: minimal labwc desktop with Noctalia'
arch=(any)
url='https://github.com/grigio/labctalia-session'
license=(MIT)
# labwc is the compositor. labwc-blur provides the same binary plus
# ext-background-effect-v1 and satisfies this dependency; the default rc.xml
# uses its <blur> block. The portals are explicit because labctalia-portals.conf
# selects wlr for screenshots/screencast and gtk for the desktop UI. polkit
# ships polkit-agent-helper-1 (Noctalia's built-in polkit agent); gnome-keyring
# provides the Secret portal; wireplumber is the PipeWire session manager and
# ships wpctl for the volume keys.
depends=(
  labwc
  noctalia
  xdg-desktop-portal-wlr
  xdg-desktop-portal-gtk
  grim
  slurp
  polkit
  gnome-keyring
  wireplumber
)
optdepends=(
  'labwc-blur: ext-background-effect-v1 blur used by the default rc.xml'
  'satty: annotate screenshots in labctalia-screenshot'
  'wl-clipboard: copy screenshots when satty is not installed'
  'ttf-inter: font used by the macos-dark theme'
  'kanshi: dynamic display profiles'
  'brightnessctl: brightness keys in the default rc.xml'
  'playerctl: media keys in the default rc.xml'
  'python: theme sync and 4-finger gestures'
  'wtype: key injection for the 4-finger gestures'
  'libinput: input reading for the 4-finger gestures'
  'polkit-gnome: fallback polkit agent when the Noctalia agent is disabled'
  'xorg-xwayland: run X11 applications (the default rc.xml sets xwayland no)'
  'gvfs: trash, recent files and volume monitoring in file choosers'
)
install=labctalia-session.install
# Keep local edits on upgrade instead of silently replacing them. labwc reads
# these only when there is no ~/.config/labwc/rc.xml / menu.xml.
backup=('etc/xdg/labwc/rc.xml' 'etc/xdg/labwc/menu.xml')
source=()
sha256sums=()

package() {
  # The repo layout mirrors /, and git preserves the executable bit on the
  # scripts. Copying the tree as-is means a file added under files/ cannot be
  # forgotten here.
  cp -a "$startdir/files/." "$pkgdir/"

  # A local `python -m py_compile` (or the CI lint job) must not leak
  # __pycache__ into the package.
  find "$pkgdir" -type d -name __pycache__ -prune -exec rm -rf -- {} +

  # The package installs the helper under a name that makes sense once it is
  # on $PATH for an admin.
  install -Dm755 "$startdir/scripts/set-default-greetd.sh" \
    "$pkgdir/usr/bin/labctalia-session-default"

  install -Dm644 "$startdir/README.md" "$pkgdir/usr/share/doc/$pkgname/README.md"
  install -Dm644 "$startdir/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
