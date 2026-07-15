# Pi Studio v1.2 — Release Notes
*Orange Pi 5B · Ubuntu 24.04 LTS (Noble) · arm64 · shipped Jul 2026 · boot-tested on hardware*

v1.2 is a **polish-and-patch** release: more studio tools, low latency made *system-wide*, a one-click
streaming launcher, a desktop refresh, and a real security pass on both userspace and the kernel.
*noisy-but-honest as always — what changed, and what it doesn't fix.*

---

## 🎹 More studio
- **LSP Plugins** — the **mixing / mastering backbone**: EQ, compressor, gate, limiter, meters (LV2,
  hosted in Ardour). Turns the stack from "make sounds" into "mix a track."
- **Audacity** — wave editor / recorder / format converter.
- **qsynth now makes sound out of the box** — ships with the **FluidR3 GM** soundfont (it was silent in
  v1.1). qsynth + VMPK = instant General-MIDI piano.
- **The Hendrix chord** 🎸 — Sonic Pi gets upstream's brand-new chords & scales (vendored same-day from
  Sam Aaron's fix): `chord(:e3, "7+9")` = the 7♯9 "Purple Haze" voicing, plus `:maj13`, the
  minor-major7 ("James Bond") family, fixed `9+5`/`m9+5`, and modal scales `:lydian_dominant`,
  `:altered`, `:phrygian_dominant`, `:double_harmonic`.

These join the existing studio (Ardour, Hydrogen, Surge XT, Yoshimi, amsynth, qsynth, the
synthv1/padthv1/samplv1/drumkv1 family, Calf / x42 / ZAM / DPF / MDA plugins, Dragonfly reverb, VMPK,
qpwgraph) — all on PipeWire.

## ⚡ Low latency — now system-wide
v1.1 went low-latency only when an app *asked*. v1.2 sets a **system PipeWire default**: quantum **256
(~5.3 ms)**, floor **64 (~1.33 ms, hardware-validated, 0 xruns under load)**. So Sonic Pi one-liners,
qsynth, and even browser audio feel tight *by default* — not just DAWs. Plus **USB autosuspend disabled
kernel-wide** (`usbcore.autosuspend=-1`) so class-compliant interfaces never drop mid-session.

## 📺 One-click Streaming
A **Streaming** launcher now lands in the dock — opens Chromium pre-configured for DRM services. (720p,
Widevine L3 — see KNOWN-ISSUES **STREAM-1**.)

## 🎨 Desktop refresh ("Sandi's Cut")
New armed-DJ-penguin wallpaper with **Technics-style decks**; terminal sized **90×38** and centered to
the wallpaper horizon; desk / Π folder icons; Home icon parked on the right edge on any resolution;
**4 new beginner Sonic Pi demos** (including the runnable wallpaper-laptop riff).

## 🩹 Fixed
- **First-boot "package removal error" (v1.1 bug):** the OEM setup's self-cleanup left a dangling
  `update-initramfs` diversion behind (casper), wedging `initramfs-tools` half-configured on every
  flashed v1.1 image. v1.2 images ship with the diversion resolved at build time.
  *(On an existing v1.1 install:* `sudo dpkg-divert --rename --remove /usr/sbin/update-initramfs
  && sudo dpkg --configure -a` *fixes it in place.)*

## 🔒 Security
- **Userspace:** built from the **current Noble archive** at bake time — picks up the June/July CVE
  stream (OpenSSL, curl, libxml2, NSS, systemd, and the rest).
- **Kernel:** **12 cherry-picked high-severity fixes** — Bluetooth (L2CAP/HIDP use-after-free,
  null-ptr, MGMT validation) and USB-audio (mixer UAF, PCM/scarlett2 overflow, UAC3 validation) — plus
  local-root ptrace hardening that was already shipped. The held BSP kernel is EOL upstream; we patch it
  by hand. **The full, honest posture is in KNOWN-ISSUES → [KERN-1](KNOWN-ISSUES.md).**

## ⚠️ Known issue shipping with v1.2
- **BT-2:** on boards with the AP6275P combo chip, **Bluetooth can fail to come up at boot when WiFi
  is active** (WiFi init races the BT firmware load — A/B-confirmed). *Workaround: on a wired
  connection, turn WiFi off and reboot — BT then comes up clean every boot.* Fix is queued for the
  next build. Full detail: [KNOWN-ISSUES → BT-2](KNOWN-ISSUES.md).

## Flagship unchanged
Still **Sonic Pi 5.0.0-beta4**. v5.0.0 *final* is a few weeks out (it brings the native theme selector);
it'll fold into a later image — v1.2 rides beta4 so it ships on time.

---

## Download & flash
Same two-part flow as v1.1 (GitHub caps release files at 2 GB), with the v1.2 filenames:
```bash
cat pi-studio-v1.2-orangepi-5b.img.xz.part* > pi-studio-v1.2-orangepi-5b.img.xz
sha256sum -c pi-studio-v1.2-orangepi-5b.img.xz.sha256    # must print: OK
# flash with balenaEtcher, or:
xz -dc pi-studio-v1.2-orangepi-5b.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
```

*Built on the board everyone says is a pain — because it can do more than they think.* 🐧🎶
