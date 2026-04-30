# Reece's Homelab
Documenting my homelab process as I go. Including set-up processes and code snippets.

## Hardware

I bought most my hardware in April 2025 before the world went insane and RAM prices skyrocketed. The 16GB Raspberry Pi 5 was $119.99 and 8GB one was $79.99. As of May 2026, those prices are now $299.99 and $169.99 respectively. 

### Integrated

- 1x Raspberry Pi 5 16GB RAM | [MicroCenter](https://www.microcenter.com/product/702590/raspberry-pi-5?rd=1)
- 1x Raspberry Pi 5 8GB RAM | [MicroCenter](https://www.microcenter.com/product/673711/5;_ARM_Cortex_A76_Quad_Core_Processor;_8GB_LPDDR4X_RAM?MccGuid=4527d294-39ed-4401-8d14-95fe686df1a3&utm_source=instorereceipt&utm_medium=email&utm_campaign=T0048&utm_content=20250405)
- 2x Raspberry Pi 5 Active Cooler | [MicroCenter](https://www.microcenter.com/product/671930/5_Active_Cooler?MccGuid=4527d294-39ed-4401-8d14-95fe686df1a3&utm_source=instorereceipt&utm_medium=email&utm_campaign=T0048&utm_content=20250411)
- 2x Inland TN320 256GB SSD NVMe PCIe Gen 3.0x4 M.2 2280 3D NAND TLC Internal Solid State Drive | [MicroCenter](https://www.microcenter.com/support/661858/TN320_256GB_SSD_NVMe_PCIe_Gen_30x4_M2_2280_3D_NAND_TLC_Internal_Solid_State_Drive?MccGuid=4527d294-39ed-4401-8d14-95fe686df1a3&utm_source=instorereceipt&utm_medium=email&utm_campaign=T0048&utm_content=20250405)
- 1x 52Pi M.2 NVMe M-KEY PoE+ Hat for Raspberry Pi 5 | [MicroCenter](https://www.microcenter.com/support/688932/M2_NVME_M-KEY_PoE_Hat_for_Raspberry_Pi_5?MccGuid=4527d294-39ed-4401-8d14-95fe686df1a3&utm_source=instorereceipt&utm_medium=email&utm_campaign=T0048&utm_content=20250405)
- 1x TP-LINK TL-SG605P 5-Port Gigabit Desktop Switch with 4-Port PoE+ | [MicroCenter](https://www.microcenter.com/product/693930/TL-SG605P_5-Port_Gigabit_Desktop_Switch_with_4-Port_PoE?MccGuid=4527d294-39ed-4401-8d14-95fe686df1a3&utm_source=instorereceipt&utm_medium=email&utm_campaign=T0048&utm_content=20250405)
- 1x Inland 1 Ft. CAT 6 Snagless, Cross UTP, Bare Copper Ethernet Cables 5-Pack - White | [MicroCenter](https://www.microcenter.com/product/638732/1_Ft_CAT_6_Snagless,_Cross_UTP,_Bare_Copper_Ethernet_Cables_5-Pack_-_White?MccGuid=4527d294-39ed-4401-8d14-95fe686df1a3&utm_source=instorereceipt&utm_medium=email&utm_campaign=T0048&utm_content=20250405)
- 1x Gateway eero Pro 6E | (Comes with my Frontier WiFi Plan)

### To Do

- 1x NVIDIA GeForce RTX 2070 Super | Got it from a co-worker

## Raspberry Pi's Initial Set-Up

### Hardware Set-Up

will fill in more later...

### OS Set-Up for NVMe SSD

#### Download Latest Pi-OS Machine Image

Downloading my latest version of Pi OS Lite based on Debian Trixie.
```bash
curl -L -o /tmp/raspios.img.xz https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2026-04-21/2026-04-21-raspios-trixie-arm64-lite.img.xz
```

Then sync OS image with NVMe SSD volume:
```bash
xzcat /tmp/raspios.img.xz | sudo dd of=/dev/nvme0n1 bs=4M status=progress
```

#### Set Up Boot Partition

Now we need to run so

Mount the boot partition first.
```bash
sudo mkdir -p /mnt/ssdboot
sudo mount /dev/nvme0n1p1 /mnt/ssdboot
```

Then enable SSH (I only plan to access my Pi's via SSH).
```bash
sudo touch /mnt/ssdboot/ssh
```

Then I created my user account, replacing `mypassword` here with my actual password.
```bash
echo "reece:$(echo 'mypassword' | openssl passwd -6 -stdin)" | sudo tee /mnt/ssdboot/userconf.txt
```

I'm using Ethernet so I don't need to transfer over my network config. 

Then I unmounted the boot volume.
```bash
sudo umount /mnt/ssdboot
```

#### Set Up Root Partition

I need to set up SSH configurations inside my root partition so that I can SSH successfully into the Raspberry Pi when officially transferring my boot partition and moving from the SD Card to the NVMe SSD drive.

```bash
sudo mkdir -p /mnt/ssdroot
sudo mount /dev/nvme0n1p2 /mnt/ssdroot
```

Then I migrated my existing SSH configurations over.
```bash
sudo mkdir -p /mnt/ssdroot/home/reece/.ssh
sudo cp ~/.ssh/authorized_keys /mnt/ssdroot/home/reece/.ssh/authorized_keys
sudo chmod 700 /mnt/ssdroot/home/reece/.ssh
sudo chmod 600 /mnt/ssdroot/home/reece/.ssh/authorized_keys
```

I need to set up the ownership temporarily to match the UID the account `reece` will have on the new install.
```bash
sudo chown -R 1000:1000 /mnt/ssdroot/home/reece/.ssh
```

Now finally unmounting the root volume.
```bash
sudo umount /mnt/ssdroot
```

#### Set Boot Order for NVMe

Now I run this command to bring up the Raspberry Pi TUI config editor.
```bash
sudo raspi-config
```

Go to **Advanced Options -> Boot Order** and select **NVMe/SSD Boot**.

Then, when prompted to reboot, reboot the Pi.

#### Home Run Stretch

When SSH'ing into the newly booted Raspberry Pi, you'll notice that it will fail because the static hostname was changed to `raspberrypi` by default. So I SSH'd into it using the following command, using my private key file on my Mac.

```bash
ssh -i ~/.ssh/<PRIVATE_KEY_FILENAME> reece@raspberrypi.local -v 
```



Check to verify that the NVMe/SSD Boot was successfully configred.
```bash
lsblk
```

Output should look something like this.
```bash
reece@raspberrypi:~$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0     2G  0 loop
mmcblk0     179:0    0 476.7G  0 disk
├─mmcblk0p1 179:1    0   512M  0 part
└─mmcblk0p2 179:2    0 476.2G  0 part
zram0       254:0    0     2G  0 disk [SWAP]
nvme0n1     259:0    0 238.5G  0 disk
├─nvme0n1p1 259:3    0   512M  0 part /boot/firmware
└─nvme0n1p2 259:4    0   238G  0 part /
```

To fix up the hostname issue, run these commands.
```bash
sudo hostnamectl set-hostname data-plane-pi-0
# Wait a minute
sudo sed -i 's/raspberrypi/data-plane-pi-0/g' /etc/hosts
```

Then reboot so that this new static hostname takes effect.
```bash
sudo reboot
```

Now to SSH back in, run using the fixed hostname again from your device (I do it with my Mac).
```bash
ssh -i ~/.ssh/<PRIVATE_KEY_FILENAME> reece@data-plane-pi-0.local -v 
```

To ensure my root partition is using the 235GB available storage instead of 3GB, I expanded the filesystem.

```bash
sudo raspi-config
```

Navigate to **Advanced Options -> Expand Filesystem**, then reboot again when prompted. To check if changes take effect when back SSH'd into the Pi, run this command.
```bash
df -h /
```

An output indicates success:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p2  235G  4.1G  221G   2% /
```

Finally, run a system upgrade.
```bash
sudo apt update && sudo apt full-upgrade -y
```

### Note
Now, I repeat the above steps for my 2nd Raspberry Pi. This way, I have faster IOPS and a bigger storage drive for each of my Pi's.
