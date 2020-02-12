# Linux for Surface gen7 (2019) (Surface Pro 7, Surface Laptop 3, ~Surface Pro X~)

Tested on Arch Linux + Surface Laptop 13" (Intel). Everything apart from
Touchscreen is working.

| Feature             | Status                                                      | Module           |
| ---                 | ---                                                         | ---              |
| Intel graphics      | Yes                                                         | i915             |
| Wireless networking | Yes                                                         | iwlwifi          |
| Bluetooth           | Yes                                                         |                  |
| Keyboard            | Yes (also backlight and caps lock lights)                   | surface_sam      |
| Touchpad            | Yes                                                         | surface_sam      |
| Touchscreen         | No (IPTS needs Intel to update drivers with GuC submission) |                  |
| Cameras             | Yes!                                                        | uvcvideo         |
| Audio               | Yes                                                         |                  |
| Microphone          | Yes                                                         |                  |
| Suspend             | Yes (s2idle/s0ix)                                           |                  |
| Power button        | Yes (requires 5.4+ kernel)                                  | soc_button_array |
| Battery status      | Yes                                                         | surface_sam      |
| Ambient light sensor| No (some debugging required)                                |                  |
| Performance modes   | ? (testing required)                                        | surface_sam      |

## Installation

**This guide is mostly obsolete. Please follow the following guide:**

https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup

## Advanced setup

Since 5.5, a custom patched kernel is mostly not required for gen7 devices
(unless you want touchscreen support).

You can skip the custom kernel installation, install kernel 5.5+, and instead
build and install the [surface-aggregator-module](https://github.com/linux-surface/surface-aggregator-module)
via dkms.

Arch Linux:
```
git clone --depth 1 --branch feature/pkg-repo https://github.com/linux-surface/linux-surface
cd linux-surface/pkg/arch/surface-aggregator-module
makepkg -s
sudo pacman -U *.pkg.tar.*
```

Surface Laptop users: until the ACPI module is installed and loaded, the touchpad and keyboard won't
function, so you'll need an external keyboard.

