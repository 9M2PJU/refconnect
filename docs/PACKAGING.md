# Linux packaging

## Local builds

```sh
# needs Go + Xorg/Wayland dev libs (see Makefile linux-packages)
make build
make deb        # -> refconnect_<version>_<arch>.deb
make rpm        # -> refconnect_<arch>.rpm
make appimage   # -> refconnect_<version>_<arch>.AppImage (needs appimagetool)
```

`VERSION` defaults to `version.json` without the leading `v`;
`GOARCH` defaults to `go env GOARCH`. Override: `VERSION=0.14.0 GOARCH=arm64 make deb`.

Install / verify:

```sh
sudo dpkg -i refconnect*.deb        # Debian/Ubuntu
sudo rpm -i refconnect*.rpm          # Fedora/RHEL (or dnf install)
sudo dnf install ./refconnect*.rpm
chmod +x refconnect*.AppImage && ./refconnect*.AppImage
dpkg -c refconnect*.deb
rpm -qlp refconnect*.rpm
```

Files installed: `/usr/bin/refconnect`, `/usr/share/applications/refconnect.desktop`,
`/usr/share/icons/hicolor/512x512/apps/refconnect.png`.

## Releases

`.github/workflows/release.yml` builds on tag `v*`:

* `build-linux (amd64)`: `tar.gz` + `.deb` + `.rpm` + `.AppImage`
* `build-linux-arm64`: `tar.gz` + `.deb` + `.rpm` + `.AppImage`
* macOS / Windows unchanged.

## AUR

Two AUR packages, maintained in separate AUR repos (not in this repo):

* `refconnect-git` (`packaging/aur/refconnect-git/PKGBUILD`) — builds from git HEAD.
* `refconnect-bin` (`packaging/aur/refconnect-bin/PKGBUILD`) — repackages the release `tar.gz`.

Publish / update:

```sh
# first time
git clone ssh://aur@aur.archlinux.org/refconnect-git.git
cp packaging/aur/refconnect-git/PKGBUILD refconnect-git/
cd refconnect-git && makepkg --printsrcinfo > .SRCINFO && git add PKGBUILD .SRCINFO && git commit -m "init" && git push

# new release (bin): bump pkgver, refresh sums
makepkg -g   # paste sha256sums_* into PKGBUILD
makepkg --printsrcinfo > .SRCINFO && git commit -am "vX.Y.Z" && git push
```

Install: `yay -S refconnect-git` or `yay -S refconnect-bin`.

## Self-updater behavior

`internal/update.ManagedInstall()` detects `/usr/bin`, `/opt`, `dpkg -S`, `rpm -qf`,
`pacman -Qo`. On managed installs `Install()` refuses and the UI (`internal/ui/updater.go`)
tells the user to use `apt`/`dnf`/`pacman` or re-download the AppImage instead of
overwriting a packaged binary.
