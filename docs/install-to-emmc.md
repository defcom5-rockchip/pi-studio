# Moving Pi Studio from SD card to eMMC (or NVMe)

Running from a good SD card is fine — but the Orange Pi 5B's onboard eMMC is faster and
frees the slot. Pi Studio ships the installer for this (inherited from Joshua-Riek's
ubuntu-rockchip): **it clones the currently running system**, so do this *after* you've
finished first-boot setup — everything you've configured comes along.

## 1 · Boot from the SD card you want to migrate

Set the system up the way you want it (user, WiFi, tweaks). The clone is a snapshot of *now*.

## 2 · Identify the eMMC

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
```

The eMMC is the `mmcblk` device that has **`boot0`/`boot1` sub-devices** (e.g. `mmcblk0`,
`mmcblk0boot0`, `mmcblk0boot1`) — that's the tell. Your SD card is the other `mmcblk` (the
one mounted as `/`). **Double-check: the target's contents will be erased.**

## 3 · Run the installer

```bash
sudo ubuntu-rockchip-install /dev/mmcblk0     # ← your eMMC device from step 2
```

It will confirm, partition the eMMC (GPT, single ext4 labeled `desktop-rootfs`), then
rsync the whole running system across (a few minutes on eMMC) and install the bootloader.
Safety note: it refuses to run against the disk you're currently booted from.

## 4 · Shut down, pull the SD card, power on

RK3588 boot order tries SD before eMMC — with the card removed, the board boots your
new eMMC system. First boot grows the filesystem to fill the eMMC automatically.

## 5 · Verify

```bash
findmnt /            # SOURCE should be the eMMC partition (e.g. /dev/mmcblk0p1)
```

Keep the SD card as-is for a while — it's a free known-good fallback. (Booting with it
inserted returns you to the SD system; nothing on the eMMC is touched.)

## Notes

- **NVMe works too**: same command with `/dev/nvme0n1` — but the SPI bootloader situation
  varies by setup; eMMC is the simple, always-works path on the 5B.
- The clone carries *everything*, including your held kernel packages and the BT/audio
  firmware fixes — no re-application needed.
- Re-running the installer later re-clones the running system (it always erases the target
  first).
