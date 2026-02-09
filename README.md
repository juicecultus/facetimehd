# facetimehd

Reverse engineered Linux driver for the FacetimeHD (Broadcom 1570) PCIe webcam found in recent MacBooks.

Based on the original [patjak/facetimehd](https://github.com/patjak/facetimehd) driver. See the upstream [Wiki](https://github.com/patjak/bcwc_pcie/wiki) for additional details.

> **This driver is experimental. Use at your own risk.**

## Supported Hardware

| Model | Identifier | Status |
|---|---|---|
| MacBook 12" (2016) | MacBook9,1 | Supported |
| MacBook 12" (2017) | MacBook10,1 | Supported |
| MacBook Pro / Air (various) | Various | See upstream wiki |

## Prerequisites

Install build tools, DKMS, kernel headers, and firmware extraction dependencies.

### Arch Linux

```bash
sudo pacman -S dkms gcc linux-headers make git cpio curl xz gzip
```

### Fedora

```bash
sudo dnf install dkms gcc kernel-devel make git cpio curl xz gzip
```

### Ubuntu / Debian

```bash
sudo apt install dkms gcc linux-headers-generic make git cpio curl xz-utils gzip
```

## Step 1 — Extract and Install Firmware

The camera requires proprietary firmware extracted from the macOS driver. This is handled by the [facetimehd-firmware](https://github.com/patjak/facetimehd-firmware) tool.

```bash
git clone https://github.com/patjak/facetimehd-firmware.git
cd facetimehd-firmware
make
sudo make install
```

This downloads the macOS driver package, extracts firmware version 1.43.0, and installs it to `/usr/lib/firmware/facetimehd/`.

## Step 2 — Clone This Repository

```bash
git clone https://github.com/juicecultus/facetimehd.git
cd facetimehd
```

## Step 3 — Build and Install via DKMS (Recommended)

DKMS will automatically recompile the module whenever you update your kernel.

```bash
sudo ln -sfn "$(pwd)" /usr/src/facetimehd-0.6.13
sudo dkms install facetimehd/0.6.13
```

### Alternative — Manual Build (without DKMS)

If you prefer not to use DKMS:

```bash
make
sudo make install
sudo depmod -a
```

> **Note:** With a manual install you will need to rebuild the module every time your kernel updates.

## Step 4 — Load the Module

```bash
sudo modprobe facetimehd
```

Verify the module is loaded and the video device exists:

```bash
lsmod | grep facetimehd
ls -la /dev/video*
```

You should see `facetimehd` in the module list and `/dev/video0` (or similar) created.

## Step 5 — Test the Camera

```bash
# Using mpv
mpv av://v4l2:/dev/video0

# Or using ffplay
ffplay /dev/video0

# Or open any webcam app (Cheese, OBS, etc.)
```

## Auto-Load on Boot

To have the module load automatically at boot, create a config file:

```bash
echo "facetimehd" | sudo tee /etc/modules-load.d/facetimehd.conf
```

## Uninstalling

### DKMS

```bash
sudo dkms remove facetimehd/0.6.13 --all
sudo rm /usr/src/facetimehd-0.6.13
```

### Manual

```bash
sudo rm /lib/modules/$(uname -r)/extra/facetimehd.ko*
sudo depmod -a
```

## Troubleshooting

- **No `/dev/video0`** — Check `dmesg | grep facetimehd` for errors. Ensure firmware is installed in `/usr/lib/firmware/facetimehd/`.
- **Corrupted image in Cheese** — Lower the resolution in Cheese preferences, then power off (not reboot) and power on.
- **Module fails to build** — Ensure `linux-headers` matches your running kernel (`uname -r`).
- **Conflict with `bdc_pci`** — The DKMS config blacklists `bdc_pci` automatically. If installed manually, add `blacklist bdc_pci` to `/etc/modprobe.d/facetimehd.conf`.
- **Sensor calibration** — The camera works without calibration files but may produce slightly incorrect colors. See [Extracting the sensor calibration files](https://github.com/patjak/bcwc_pcie/wiki/Extracting-the-sensor-calibration-files) for details.

## DKMS Status Check

```bash
dkms status
```

Expected output:

```
facetimehd/0.6.13, <kernel-version>, x86_64: installed
```

## License

See [LICENSE](LICENSE) for details.
