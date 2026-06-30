# Pi Studio — Known Issues

An honest register of real, verified-on-hardware limitations that ship with the image. We document
these instead of hiding them — cause, workaround, and whether a fix is planned.

---

## BT-1 — Leave Bluetooth ON; toggling it OFF needs a reboot
**Severity:** low–medium · **Status:** root-caused + mitigated; fuller fix on the post-launch list.

Bluetooth comes up working on **every boot** — pair, connect, stream audio. But if you switch the
Bluetooth **adapter** fully OFF (the master toggle, Airplane Mode, or `rfkill`), it won't come back
on without a **reboot**.

**Why:** it's a side-effect of Pi Studio's **low-latency kernel**, not a hardware defect — the same
board recovers fine on a stock-preemption kernel. The Wi-Fi/BT combo chip reloads its firmware over a
timing-sensitive UART sequence at power-on; full preemption + 1000 Hz disrupts the *live* re-init
(boot is fine, off→on is not).

**Rule:** once Bluetooth is on, leave the adapter on. Pair/connect freely — just don't flip the master
switch off. If it does go off, reboot.

**Mitigation shipped:** the easiest way to trip this is mis-tapping the off button next to the
device-select arrow, so the desktop theme adds a safety **gap** between them (credit: **sandi-pi**).

**Planned fix:** run the Bluetooth bring-up at realtime priority so preemption can't interrupt it,
and/or offer a stock-preempt kernel as a boot-menu fallback.

---

## DISP-1 — Use 4K@60 for windowed work; 4K@120 for fullscreen
**Severity:** low · **Status:** known VOP2 limit, workaround available.

At 4K@**120**, *windowed* GPU-composited content (browsers, apps) can flicker. Fullscreen video is
fine (it bypasses the compositor). Run the desktop at **4K@60** (Settings → Displays → Refresh Rate)
for everyday use; use 4K@120 for fullscreen video/motion. It's the VOP2 compositor frame-timing
ceiling, not a decode or browser bug.

---

## BROW-1 — Chromium is the hardware-accelerated browser
**Severity:** low · **Status:** fixed via launcher flag.

Use **Chromium** (the rkmpp build) for hardware video decode and clean rendering — it's the designated
primary browser. **Vivaldi** runs under XWayland on this board's immature GPU stack and software-
decodes video, so it's launched with `--disable-gpu` for a clean software render.

---

## STREAM-1 — DRM streaming video tops out at 720p
**Severity:** low · **Status:** platform limitation (Widevine L3 on Linux).

DRM video plays at **720p maximum** — the Widevine **L3** ceiling that applies to *all* arm64 Linux
browsers, not a Pi Studio limit (1080p/L1 needs hardware key paths unavailable here). Pi Studio ships
an on-desktop **Widevine setup guide**; once enabled, DRM services play in Chromium (Prime Video,
Disney+, Spotify, DRM YouTube). Protected streams are software-
decoded (normal on Linux) and play smoothly at 720p.

---
*New verified-on-hardware limitations get added here. Honesty > hiding.*
