# libfprint-ft9366 (Arch package)

libfprint with an open-source driver for the **FocalTech FT9366 fingerprint sensor**
(USB `2808:a658`, Realtek RTS5811 bridge, ASUS VivoBook and others).

```sh
git clone https://github.com/tattwamashi/libfprint-ft9366-pkg
cd libfprint-ft9366-pkg
makepkg -si
fprintd-enroll
```

It builds the official libfprint release with every upstream driver plus the ft9366 driver
(`ft9366-driver.patch`), using Arch's build options, and replaces the `libfprint` package, so
fprintd works unchanged. A prebuilt package is attached to each
[release](https://github.com/tattwamashi/libfprint-ft9366-pkg/releases):

```sh
sudo pacman -U https://github.com/tattwamashi/libfprint-ft9366-pkg/releases/download/1.94.100-1/libfprint-ft9366-1.94.100-1-x86_64.pkg.tar.zst
```

To go back to the stock library: `sudo pacman -S libfprint`.

Driver source, documentation and issues: https://github.com/tattwamashi/libfprint
An AUR package of the same name will follow once AUR registration reopens.

License: LGPL-2.1-or-later.
