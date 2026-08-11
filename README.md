# pcoip-client (Arch Linux PKGBUILD)

Community packaging for **HP Anyware Client** (formerly *Teradici PCoIP Client*) on Arch Linux by repackaging the official Ubuntu `.deb`.

- This is **not** an official HP/Teradici project.
- The client is **proprietary**; you must comply with the upstream license/EULA.
- `x86_64` only.

## Packages

This PKGBUILD produces split packages:

- `pcoip-client`: the main GUI client (`/usr/bin/pcoip-client`)
- `pcoip-client-clipboard`: optional clipboard sync vchan plugin

## Install (build from this repo)

```bash
makepkg -si
```

## Notes

- The PKGBUILD fetches the upstream client `.deb`, the Ubuntu `libprotobuf32` it expects for ABI compatibility, and the Qt 6.9.2 `libQt6WaylandClient`/`libQt6WaylandEglClientHwIntegration` from the Arch package archive.
- Main package depends on system Qt, X11/XCB, Wayland, audio, and VA-API; optional VA-API drivers are surfaced via `optdepends`.
- The wrapper prefers native Wayland when `WAYLAND_DISPLAY` is set and falls back to X11/XCB. The bundled Wayland Qt platform plugins are kept and pointed at the vendored Qt 6.9 Wayland client libs so they load against the client's bundled Qt 6.9.3 (Arch's `qt6-wayland` is too new and requires `Qt_6.11` symbols).
- The clipboard plugin now pulls in `graphicsmagick` and is patched to link against the Arch SONAME.
- Capabilities for the client and USB helper are baked into the package during build; if they get lost, re-apply them manually (see troubleshooting).
- The upstream `.desktop` file is patched to avoid registering the `pcoip://` URL handler (to prevent collisions with dedicated URL-handler packages).

## libva Driver

The client uses VA-API for hardware-accelerated video decoding during remote sessions.
Without a driver, you'll see this warning in logs — decoding falls back to software
(works, but uses more CPU):

```
[AVHWDeviceContext] Failed to initialise VAAPI connection: -1 (unknown libva error).
```

Install the driver matching your GPU. To check which GPU you have:

```bash
lspci | grep -iE "vga|3d|display"
```

| GPU | Driver | Install |
|-----|--------|---------|
| Intel (Skylake 2015 or newer) | `intel-media-driver` | `sudo pacman -S intel-media-driver` |
| Intel (Broadwell 2014 or older) | `libva-intel-driver` | `sudo pacman -S libva-intel-driver` |
| AMD / Intel via Mesa | `libva-mesa-driver` | `sudo pacman -S libva-mesa-driver` |
| NVIDIA (proprietary driver) | `libva-nvidia-driver` | `sudo pacman -S libva-nvidia-driver` |

These are listed as `optdepends` and are not required — the client runs fine without them.

## Troubleshooting

- **USB redirection not working / permission errors**
  - Re-apply capabilities:
    - `sudo setcap cap_setgid+p /usr/libexec/pcoip-client/pcoip-client`
    - `sudo setcap cap_setgid+i /usr/libexec/pcoip-client/usb-helper`
- **Missing shared libraries**
  - Check: `ldd /usr/bin/pcoip-client | rg 'not found'`
- **ERROR: Cannot find the fakeroot binary.**
  - Check you have the base-devel package installed: `sudo pacman -S base-devel`
