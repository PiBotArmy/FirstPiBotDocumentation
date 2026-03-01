# Raspberry Pi 5 + 52Pi 3.5" SPI Touchscreen

## Final Working Configuration Reference

## Goal

Run Raspberry Pi OS Desktop on the 52Pi 3.5" SPI display (ILI9486)\
and toggle between SPI and HDMI as the primary display.

------------------------------------------------------------------------

## System Requirements

-   Raspberry Pi 5\
-   Raspberry Pi OS (Bookworm) 64-bit with Desktop\
-   X11 session (not Wayland)\
-   52Pi 3.5" SPI touchscreen (ILI9486)

------------------------------------------------------------------------

## Required `/boot/firmware/config.txt` Settings

Under `[all]`:

``` ini
dtparam=spi=on
dtoverlay=vc4-kms-v3d
dtoverlay=piscreen,drm,speed=32000000,rotate=0
max_framebuffers=1
```

### Notes

-   No spaces after commas in the `dtoverlay` line.
-   Do **NOT** use LCD-show on Raspberry Pi 5.
-   Do **NOT** duplicate `ads7846` overlay when using `piscreen`.

------------------------------------------------------------------------

## Identify DRM Devices

List DRM cards:

``` bash
ls -l /dev/dri
```

Identify driver bindings:

``` bash
for c in /sys/class/drm/card*/device/driver; do
  echo "== $c =="
  readlink -f "$c"
done
```

Expected: - `vc4-drm` → HDMI GPU\
- `ili9486` → SPI display (example: card2)

------------------------------------------------------------------------

## SPI Display Toggle Script

File: `/usr/local/bin/use-spi`

``` bash
#!/bin/bash

echo "Detecting SPI (ili9486) DRM device..."

SPI_CARD=$(for c in /sys/class/drm/card*/device/driver; do
    if readlink -f "$c" | grep -q ili9486; then
        basename $(dirname $(dirname $c))
    fi
done)

if [ -z "$SPI_CARD" ]; then
    echo "ERROR: Could not detect ili9486 DRM device."
    exit 1
fi

echo "SPI device found: /dev/dri/$SPI_CARD"

sudo mkdir -p /etc/X11/xorg.conf.d

cat <<EOF | sudo tee /etc/X11/xorg.conf.d/99-display.conf
Section "Device"
    Identifier  "SPI-ILI9486"
    Driver      "modesetting"
    Option      "kmsdev" "/dev/dri/$SPI_CARD"
EndSection
EOF

echo "Done. Reboot required."
```

Make executable:

``` bash
sudo chmod +x /usr/local/bin/use-spi
```

Use:

``` bash
sudo use-spi
sudo reboot
```

------------------------------------------------------------------------

## HDMI Display Toggle Script

File: `/usr/local/bin/use-hdmi`

``` bash
#!/bin/bash

echo "Switching to HDMI display..."

sudo rm -f /etc/X11/xorg.conf.d/99-display.conf

echo "Done. Reboot required."
```

Make executable:

``` bash
sudo chmod +x /usr/local/bin/use-hdmi
```

Use:

``` bash
sudo use-hdmi
sudo reboot
```

------------------------------------------------------------------------

## Important Notes

-   SPI and HDMI are separate DRM devices.
-   Mirroring between HDMI and SPI is not native on Pi 5.
-   SPI screen is ideal as the primary robot UI display.
-   HDMI is best used for maintenance/debugging.
-   Reboot required after switching modes.
-   Keep configuration minimal and clean.

------------------------------------------------------------------------

## Current Stable State

-   SPI display boots into full desktop.
-   HDMI can be toggled on/off as primary display.
-   System stable under `graphical.target`.
-   Ready for robot application deployment.

------------------------------------------------------------------------

End of reference.
