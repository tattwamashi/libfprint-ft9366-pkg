# libfprint-ft9366 (Arch package)

libfprint with an open-source driver for the **FocalTech FT9366 fingerprint sensor**
(USB `2808:a658`, Realtek RTS5811 bridge, ASUS VivoBook and others).

Check that you have this sensor:

```sh
lsusb | grep 2808:a658
# ... ID 2808:a658 Realtek USB2.0 Finger Print Bridge FocalTech Fingerprint Device
```

## Install

```sh
git clone https://github.com/tattwamashi/libfprint-ft9366-pkg
cd libfprint-ft9366-pkg
makepkg -si
```

or the prebuilt package from the
[releases](https://github.com/tattwamashi/libfprint-ft9366-pkg/releases):

```sh
sudo pacman -U https://github.com/tattwamashi/libfprint-ft9366-pkg/releases/download/1.94.100-1/libfprint-ft9366-1.94.100-1-x86_64.pkg.tar.zst
```

It builds the official libfprint release with every upstream driver plus the ft9366 driver
(`ft9366-driver.patch`), using Arch's build options, and replaces the `libfprint` package.
Pacman asks to remove `libfprint`: answer yes.

## Setup after installing

### 1. fprintd

```sh
sudo pacman -S --needed fprintd
fprintd-list $USER        # should show "FocalTech FT9366 (RTS5811)"
```

fprintd starts on demand; there is no service to enable.

### 2. Enroll a finger

```sh
fprintd-enroll                         # right index finger by default
fprintd-enroll -f left-index-finger    # more fingers, if you like
fprintd-verify                         # test it
```

If `fprintd-enroll` fails with *Not Authorized* or *PermissionDenied* (common on Hyprland,
Sway and other desktops without a polkit agent), enroll as root for your user:
`sudo fprintd-enroll $USER`.

Enrollment takes 20 touches. The sensor sees only about 3×4 mm of skin, so:

- place the finger **the way you will actually touch it** when unlocking, shifting it a
  millimetre or two between touches; if it says "Move your finger slightly", move a bit more
- lift the finger completely between touches
- **touch lightly**: on many laptops the sensor is the power button, and a firm press can
  suspend or power off the machine

If you enrolled with a very different placement and verify keeps failing, re-enroll:
`fprintd-delete $USER && fprintd-enroll`.

### 3. Use it for authentication (PAM)

Add **one line** at the top of the `auth` section of each service you want, and leave the
rest as is:

```
auth      sufficient   pam_fprintd.so max-tries=3 timeout=10
```

`sufficient` means a matching finger logs you in, and a failed or timed-out scan falls
back to your password, so you can't lock yourself out. Keep a root shell open while you
edit PAM files, and test in a new terminal.

Pick the services one at a time, rather than editing `system-auth` (which affects everything,
including SSH and `su` for other users):

| What | File | How |
|---|---|---|
| `sudo` | `/etc/pam.d/sudo` | add the line at the top |
| `su` | `/etc/pam.d/su` | add the line at the top |
| GUI password prompts (polkit) | `/etc/pam.d/polkit-1` | `sudo cp /usr/lib/pam.d/polkit-1 /etc/pam.d/polkit-1` first, then add the line (edits in `/usr/lib` are overwritten by updates) |
| SDDM login screen | `/etc/pam.d/sddm` | add the line at the top, see below |
| LightDM / greetd / other | `/etc/pam.d/<name>` | same pattern |
| GNOME (GDM) | built in | Settings → Users → Fingerprint Login |
| KDE Plasma | built in | System Settings → Users → Configure Fingerprint Authentication (login and lock screen) |

Example for `sudo`:

```sh
sudo cp /etc/pam.d/sudo /etc/pam.d/sudo.bak
sudo sed -i '1a auth\t\tsufficient\tpam_fprintd.so max-tries=3 timeout=10' /etc/pam.d/sudo
sudo -k; sudo true        # in a new terminal: touch the sensor at the prompt
```

Pacman keeps your edits in `/etc/pam.d/` across package updates (new defaults arrive as
`.pacnew` files).

#### Login screen (SDDM and others)

With the line in `/etc/pam.d/sddm`, select your user and **press Enter with the password
field empty** (or click log in), then touch the sensor. Typing your password still works.

Logging in with a fingerprint can't unlock your keyring (gnome-keyring, KWallet), because
no password was given. Apps that store secrets will then ask for the keyring password. If
that bothers you, use the password at the login screen once per boot and the fingerprint
for the lock screen, sudo and polkit.

#### Lock screens

| Lock screen | Setup |
|---|---|
| hyprlock | in `~/.config/hypr/hyprlock.conf`: `auth { fingerprint { enabled = true } }` |
| swaylock | add the line at the top of `/etc/pam.d/swaylock`; press Enter on the empty field, then touch |
| GNOME, KDE | through their settings, as above |
| others | most use a PAM service named after them in `/etc/pam.d/`: same one-line change |

## Troubleshooting

- **"No devices available"**: check `lsusb | grep 2808:a658`, and that `pacman -Q libfprint-ft9366`
  shows this package (not the stock `libfprint`). Then `sudo systemctl restart fprintd`.
- **Verify fails often**: re-enroll with the placement you actually use, and touch lightly.
  A finger resting on the sensor when the scan starts is handled: lift and touch again.
- **Logs**: `journalctl -u fprintd -b`. For detailed driver logs, add
  `Environment=G_MESSAGES_DEBUG=all` to a drop-in (`sudo systemctl edit fprintd`), restart
  fprintd and try again.
- **Bug reports**: https://github.com/tattwamashi/libfprint/issues, with your laptop model and
  the log of a failed attempt.

## Uninstall

```sh
fprintd-delete $USER            # optional: remove enrolled fingers
# remove the pam_fprintd lines you added (or restore your .bak copies)
sudo pacman -S libfprint        # back to the stock library
```

## About

This is new code, tested on one laptop so far. It's convenient, but treat it as convenience
rather than a security boundary until it has seen more hardware: keep your password as the
fallback. Reports from other `2808:a658` machines are very welcome.

Driver source, documentation and issues: https://github.com/tattwamashi/libfprint
An AUR package of the same name will follow once AUR registration reopens.

License: LGPL-2.1-or-later.
