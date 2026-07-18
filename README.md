# 🐧 Pi Studio

**A low-latency audio-production Linux distribution for the Orange Pi 5B (Rockchip RK3588).**
Free, open, fully offline, and privacy-first. Flagship: **Sonic Pi** live-coding music.

*noisy-but-honest — we tell you what works, what doesn't, and why.*

---

## What it is
Pi Studio is a music-production OS for the Orange Pi 5B (Rockchip RK3588) — around $250 for the 16 GB
board. Flash this image and you've got a real studio: a kernel **re-tuned for real-time audio**, a
curated stack of open-source music tools, hardware-accelerated 4K video, and a desktop built for making
sound — not phoning home. Based on Ubuntu 24.04 (Noble), arm64.

## Highlights
- 🎚️ **Verified low-latency kernel — hit a key, hear it *now*.** Full preemption + 1000 Hz tick.
  Measured on hardware: **~115 µs worst-case scheduling latency under full 8-core load, 0 xruns at a
  1.33 ms buffer** (~82× lower worst-case jitter than a stock desktop kernel — same chip, config alone).
- 🎵 **Sonic Pi 5.0.0-beta4** — the newest upstream release — front and center, with a custom *PiStudioMode*
  theme, game-controller input, and MIDI-clock master/slave so it drives (or follows) your gear.
- 🖥️ **4K@120** display (custom-patched VOP2 kernel) + **hardware video decode** in Chromium (RK3588 rkmpp).
- 🎹 **A full open-source studio** — Ardour, Hydrogen, Rosegarden, Surge XT, Yoshimi, amsynth, qsynth,
  Calf plugins, VMPK, qpwgraph, and more, on PipeWire with realtime priority.
- 🔌 **Class-compliant USB audio** interfaces work out of the box (no autosuspend dropouts).
- 🔒 **Offline & private** — no telemetry, no phone-home, **snap-free**, works with no internet.
- 🎧 **WiiM control, built in** — drive your WiiM streamer from the desktop, out of the box. Rare in a
  Linux distro; standard here.
- 📺 **DRM streaming video at 720p** — Chromium plays Prime Video, Disney+, Spotify, and DRM YouTube;
  enable Widevine with the guide that ships on the desktop.

## What you'll hear
- **Tighter feel.** Less delay between hitting a key and the sound — monitoring and live coding feel
  immediate, not laggy.
- **Clean under load.** Render a busy `live_loop` while the system works — no clicks, no dropouts.
- **Smaller buffers.** Push the buffer down to the lowest your interface allows, and stay glitch-free.

It's not just a label: the kernel is **built from a re-tuned config** (verified `CONFIG_PREEMPT=y`,
`CONFIG_HZ=1000`), not a stock kernel wearing a new name. We traded throughput you'll never miss for
timing you'll definitely hear.

## Hardware
- **Orange Pi 5B** (Rockchip RK3588, 8-core, Mali-G610, arm64) — the primary target.
- The audio + kernel work applies to other RK3588 boards too; the 5B is what's tested and tuned.

### Moving to eMMC
Running from SD? [docs/install-to-emmc.md](docs/install-to-emmc.md) — one command clones your set-up system onto the onboard eMMC.

## Download & flash
The image ships in two parts (GitHub caps release files at 2 GB each).
1. From **[Releases](../../releases)**, download **both** parts + the checksum:
   `…img.xz.part00`, `…img.xz.part01`, `…img.xz.sha256`.
2. Rejoin and verify:
   ```bash
   cat pi-studio-v1.1-orangepi-5b.img.xz.part* > pi-studio-v1.1-orangepi-5b.img.xz
   sha256sum -c pi-studio-v1.1-orangepi-5b.img.xz.sha256   # must print: OK
   ```
3. Flash with [balenaEtcher](https://etcher.balena.io/) (on the rejoined `.img.xz`), or:
   ```bash
   xz -dc pi-studio-v1.1-orangepi-5b.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
   ```
4. Boot — you're in a tuned studio. First-boot tips land on the desktop.

## Streaming / DRM
Chromium plays DRM streaming video at **720p** — the Widevine **L3** ceiling on Linux, not a Pi Studio
limit. Enable Widevine using the **guide that ships on the desktop**; then **Amazon Prime Video,
Disney+, Spotify, and DRM YouTube** play. Protected streams are software-decoded (normal on Linux) and
play smoothly at 720p.

## ⚠️ Known issues — read them, we don't hide them
We ship an honest **[KNOWN-ISSUES](KNOWN-ISSUES.md)** register. The short version:
- **Bluetooth:** works great — just leave the adapter **on**; toggling it off needs a reboot to recover
  (low-latency-kernel + chip-firmware quirk; mitigated in the UI).
- **4K@120:** flawless full-screen; for everyday windowed use run **4K@60** (no flicker).
- **DRM video:** 720p maximum (Widevine L3 cap, Linux-wide).

## Credits
- Kernel 4K@120 patch — **jamautt**
- Bluetooth UI safety fix — **sandi-pi**
- Base image — **Joshua-Riek / ubuntu-rockchip**
- Widevine enabler — **AsahiLinux widevine-installer** (user-installed; not bundled — Widevine is Google-proprietary)
- And the upstream projects: Sonic Pi, Ardour, PipeWire, Surge XT, and the RK3588 community.

## License
See [LICENSE](LICENSE). Built from open-source components; their licenses apply.

---
*Built on the board everyone says is a pain — because it can do more than they think.* 🐧🎶
