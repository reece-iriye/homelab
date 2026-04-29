# Homelab
Documenting my homelab process as I go. Including set-up processes and code snippets.

## Hardware Set-Up

will fill in more later...

## OS Set-Up

Plug the bad boy in. 

Unsure if this was needed, but add this line to `/boot/firmware/config.txt`:
```bash

```


Downloading my latest version of Pi OS Lite based on Debian Trixie.
```bash
curl -L -o /tmp/raspios.img.xz https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2026-04-21/2026-04-21-raspios-trixie-arm64-lite.img.xz
```

Then sync OS image with NVME SSD volume:
```bash
xzcat /tmp/raspios.img.xz | sudo dd of=/dev/nvme0n1 bs=4M status=progress
```

