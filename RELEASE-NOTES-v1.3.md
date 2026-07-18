# Pi Studio v1.3 — Release Notes
*Orange Pi 5B · Ubuntu 24.04 LTS (Noble) · arm64 · shipped Jul 2026 · hardware-verified (22-point first-boot sweep)*

v1.3 is a **fix-and-grow** release: two Bluetooth bugs put to rest (one of them industry-wide),
a JACK-compatibility hole plugged, five new studio apps — including a Carla build Ubuntu itself
dropped — and a proper first-boot welcome. *Noisy-but-honest, as always.*

---

## 🩹 Fixed — update even if you want nothing new
- **JACK-era apps crashed on v1.2** (SooperLooper and friends died with `JackTemporaryException`):
  the image shipped jackd2's libjack with no PipeWire bridge. v1.3 ships **`pipewire-jack`** with
  correct linker precedence — every JACK app now lands on PipeWire automatically. *(On an existing
  v1.2 install: `sudo apt install pipewire-jack`, then copy `/usr/share/doc/pipewire-jack/examples/ld.so.conf.d/pipewire-jack-*.conf`
  to `/etc/ld.so.conf.d/00-pipewire-jack.conf` — the `00-` prefix matters — and run `sudo ldconfig`.)*
- **BT-2 FIXED — "Bluetooth greyed out at boot":** the years-old AP6275P WiFi/BT boot race is dead.
  Root-caused, A/B-validated 4/4, first-boot-proven. Public record:
  [ubuntu-rockchip#1](https://github.com/defcom5-rockchip/ubuntu-rockchip/issues/1).
- **BT-3 MITIGATED — Bluetooth audio stutter with WiFi active:** v1.3 ships tuned BT-coexistence
  firmware timing parameters — the same values Raspberry Pi deploys on millions of boards, which the
  RK3588 ecosystem never inherited. Ear-validated matrix + 14 h soak:
  [ubuntu-rockchip#2](https://github.com/defcom5-rockchip/ubuntu-rockchip/issues/2). *(Residual
  limits documented in [KNOWN-ISSUES → BT-3](KNOWN-ISSUES.md) — SBC transients, and check your WiFi
  password isn't stuck retrying.)*

## 🎛️ New in the studio
- **Carla 2.5.10** — the plugin-host Swiss-army-knife. Ubuntu Noble dropped Carla entirely;
  **we built it from source on the RK3588 and ship our own pinned deb.**
- **Guitarix** — guitar amp + pedalboard.
- **Dragonfly Reverb** — the beloved LV2 reverb family (all 4 bundles).
- **SooperLooper** — live looping (works out of the box now — see the JACK fix above).
- **DrumGizmo** — multichannel drum sampler, plus **`pistudio-get-drumkit`**: one command fetches a
  kit from the official catalog. Kits run 11 MB to 5.4 GB — that's why they're not baked in.
- Deferred honestly: **sfizz** (no Noble arm64 package → v1.4), **NeuralNote** (own build → v1.3.1).

## 👋 First-boot welcome
Setup now opens with a six-slide **Pi Studio tour** — what's installed, the latency story, our
honest KNOWN-ISSUES philosophy, and a runnable Sonic Pi demo using the `:phrygian_dominant` scale
this distro added. (Yes, the last slide has a llama.)

## 🔒 Carried forward
Everything from v1.2 rides along: the Hendrix `7+9` chord + new scales, the low-latency system-wide
PipeWire defaults, 12 cherry-picked kernel CVE fixes ([KERN-1](KNOWN-ISSUES.md) has the honest
posture), first-boot casper fix, name-clean kernel, zero snaps, zero telemetry.

**Honest line:** USB-MIDI hotplug and GUI plugin-scan acceptance are pending one desk session with a
MIDI controller; anything failing there gets the fix-or-document treatment.

---

## Download & flash
Same two-part flow (GitHub caps release files at 2 GB):
```bash
cat pi-studio-v1.3-orangepi-5b.img.xz.part* > pi-studio-v1.3-orangepi-5b.img.xz
sha256sum -c pi-studio-v1.3-orangepi-5b.img.xz.sha256    # must print: OK
xz -dc pi-studio-v1.3-orangepi-5b.img.xz | sudo dd of=/dev/sdX bs=4M status=progress conv=fsync
```

*Built on the board everyone else stopped building for.* 🐧🎶
