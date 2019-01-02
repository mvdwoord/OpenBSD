# Installation/recovery USB stick

The `install.fs` image, when written to a USB stick creates a minimally sized patrition for booting and installing. I have modified the stick to have an extra disklabel d with the remaining space. This paritiion holds everything (more or less) to do a complete *offline* re-installation in various ways.

On first boot I `mount /dev/sd2d /mnt` and perform system configuration.

## Firmware
Some firmware packages are not distributed along with OpenBSD due to licensing incompatibility. I keep a full set from the [OpenBSD Firmware Repository](http://firmware.openbsd.org/firmware/).
Install the packages offline using `fw_update -p .`

## Packages
No need to keep the entire packae repository local, however with a simple script I keep a cache of all installed packages.
```
PKGS=$(pkg_info | awk '{print $1}')
for pkg in $PKGS
        do
                if ! [ -f ./${pkg}.tgz ]; then
                        curl -o ${pkg}.tgz $(cat /etc/installurl)/6.4/packages/amd64/${pkg}.tgz
                fi
        done
```

Modify to taste.

To get a list of all installed packages, sans the dependencie, to get a minimum install command I use `pkg_info -m | grep -v 'firmware' | awk '{print $1}'`

If you set PKG_CACHE to that folder, every time you use pkg_add(1) to install something, the downloaded packages (and dependencies) will store a copy there. When you re-install, set PKG_PATH to that folder and install from local files.

For ports I copy the `distfiles`