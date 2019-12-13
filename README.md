# Linux for Surface gen7 (2019) (Surface Pro 7, Surface Laptop 3, Surface Pro X?)

Primarily tested on Arch Linux + Surface Laptop 13" (Intel). Everything apart
from Touchscreen is working.

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
| Performance modes   | ? (testing required)                                        | surface_sam      |

# Setup

A custom patched kernel build is required. The prebuilt packages are **not
recommended** because they include a legacy i915 driver to make IPTS work on
older Surface devices. This will **not work** for gen7 and will make your graphics
buggy and occasionally crash the compositor. You have been warned.

The rest of this guide will take you through setting up a minimally patched
kernel and qzed's custom ACPI module (which is necessary for keyboard, touchpad,
battery status and performance modes to function!)

You want to install **kernel 5.4+** which contains the icelake-pinctrl changes
necessary for the power button to work.

## Patch list

Included is a curated subset of patches from
[qzed/linux-surfacegen5-acpi](https://github.com/qzed/linux-surfacegen5-acpi)
and [qzed/linux-surface](https://github.com/qzed/linux-surface).

- We don't use any of the jakeday patches since they're just older versions of qzed's patches.
- None of the IPTS patches are needed since the feature doesn't work on gen7 at the moment.
- Wifi patches are unnecessary, gen7 uses intel's wifi chip instead of marvell.
- I don't recommend any of the old /etc config files because they either work
    around marvell wifi issues or solve old issues.

Depending on when you read this, you might not need some (or all) of these anymore:

| Name                                                       | Description                                                      | Upstream          |
| ----                                                       | --------                                                         | -----             |
| 0001-ACPI-Fix-buffer-integer-type-mismatch.patch           | Works around an ACPI incompatibility | No ([Possibly soon](https://patchwork.kernel.org/patch/10853017/)) |
| 0002-serdev-Add-ACPI-devices-by-ResourceSource-field.patch | Needed for ACPI detection            | Scheduled for 5.5 |
| 0004-hid.patch                                             | Needed to fix touchpad detection     | Scheduled for 5.5 |
| 0010-ioremap_uc.patch                                      | Needed to fix a CPU firmware bug     | Scheduled for 5.5 |

If things go well, we should be able to use the mainline kernel together with
the acpi module soon, which will simplify the process a lot.

## Instructions

Until the ACPI module is installed and loaded, the touchpad and keyboard won't
function, so you'll need an external keyboard.

TODO: expand with more detail

- Shrink the windows partition
- [Disable secure boot](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/disabling-secure-boot)
- [Boot into UEFI to change the boot order to allow booting from USB](https://support.microsoft.com/en-us/help/4023511/surface-boot-surface-from-a-usb-device)
- Install Linux via a bootable USB
    - If you are having issues with the boot process hanging, temporarily add
        `modprobe.blacklist=intel_lpss_pci` to the boot params. The custom
        kernel includes a patch for this issue.
    - Protip: You can use `copytoram` which will copy the resouces to RAM,
        allowing you to disconnect the USB and plug in an external keyboard.
- Build and install the custom kernel, using `.patch` files in the `kernel/` folder (Arch: a custom PKGBUILD is included: `cd kernel; MAKEFLAGS="-j4" makepkg -s --skippgpcheck; sudo pacman -U *.pkg.tar.xz`)
- Make sure to update your bootloader to point to the new kernel.
- Reboot
- Build and install the [surface acpi module](https://github.com/qzed/linux-surfacegen5-acpi) via dkms (Arch: `cd surface_acpi; makepkg -s; sudo pacman -U *.pkg.tar.xz`)

Congrats! Most things should work now. A few additional steps to address various problems:

- Add `reboot=pci` to your kernel parameters to fix `reboot` (normally hangs at the surface logo)
- If wifi fails to work, and `dmesg` output says it's failing to load ucode, [follow these instructions](https://gitlab.com/emrose/xps13-7390_debian/issues/5#note_240886447)

#### Intel CPU
- Install `intel-ucode` for latest CPU microcode.
- If using dm-crypt: Add `intel_spi_pci intel_lpss_pci xhci_pci idma64 8250_dw surface_sam?` to your mkinitcpio MODULES list to have a working keyboard at the LUKS screen.

#### AMD CPU
[please report your findings]

### Extras
- Add user to `video` to allow changing the backlight
- Possibly use libinput-gestures for touchpad gestures
- Possibly use powertop (but don't disable "PCI Device Intel Corporation Ice Lake-LP USB 3.1 xHCI Host" or keyboard/touchpad will start dropping events)
- Setup a swapfile, then enable `suspend-then-hibernate` mode
- Test the webcam via `mpv av://v4l2:/dev/video0 --demuxer-lavf-format=video4linux2 --demuxer-lavf-o-set input_format=mjpeg` (video2 for the Windows Hello IR camera)
    - it supports both YUV and MJPEG input formats, but while YUV is the default, only MJPEG is 720p

## Donations

Aim them at [qzed](https://github.com/qzed/linux-surfacegen5-acpi/blob/master/README.md#donations)
-- he maintains the ACPI module, latest patchsets and prebuilt kernels that make gen5+ viable on linux
