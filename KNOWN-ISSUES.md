# Pi Studio — Known Issues (ship-with register)

Honest list of real limitations that ship with the image. Each is verified on hardware, with the
cause, the user-facing workaround, and whether a fix exists. Don't hide these — document them.

---

## KERN-1 — The kernel is a held vendor 6.1 BSP, hand-patched for security
**Status:** By design; the honest tradeoff behind the whole distro. Review it before you decide to run Pi Studio.
**Severity:** Informational — but it IS the security posture.

**The situation:** Pi Studio's kernel is Rockchip's vendor **6.1 BSP** (Ubuntu `1027.27` base), modified
for low latency (`CONFIG_PREEMPT`, 1000 Hz) and flicker-free 4K@120 (AFBC-reject patch). That vendor line
is **EOL upstream** — there is no newer 6.1 vendor base to rebase onto, and the packages are `apt-mark hold`
so an `apt upgrade` can't silently replace the tuned kernel.

**What that means for security:** the kernel does NOT track the upstream stable CVE stream automatically.
Instead, each release **cherry-picks high-severity fixes by hand**, chosen for what an audio appliance
actually exposes: v1.2 carries **12 backported fixes** (Bluetooth L2CAP/HIDP use-after-free/null-deref/MGMT
validation; USB-audio mixer UAF, PCM/scarlett2 overflow, UAC3 validation) plus local-root ptrace hardening.
**Coverage is partial by design** — remote-network CVE classes that don't apply to a typically-offline,
firewalled studio box are consciously skipped. Userspace, by contrast, is built from the current Ubuntu
archive at bake time and patches normally via `apt` (see MAINT-1).

**Rule for users:** treat the box like the studio appliance it is — behind your router, not port-forwarded
to the internet. If your threat model needs a fully-tracked kernel, run stock Ubuntu Rockchip instead and
give up the low-latency/4K@120 tuning; that's the honest trade.

**Fix path:** the real long-term answer is the **mainline kernel** (upstream RK3588 support is close —
HDMI 2.1 FRL PHY landed in 7.0; full 4K@120 not proven yet). When mainline can do what the BSP does,
Pi Studio migrates and the cherry-pick era ends. Until then: hand-picked backports, documented per release.

---

## BT-1 — Turning Bluetooth OFF disables it until reboot (low-latency-kernel side-effect)
**Status:** Root-caused + **mitigated**; full fix on the post-launch list. **Not a chip defect, not
universal** — a side-effect of Pi Studio's low-latency kernel.
**Severity:** Low–medium — Bluetooth works fully; only the *off→on* toggle is affected, and the most
common way to trip it (an accidental mis-tap) is mitigated in the UI.
**Verified:** 2026-06-28/29 on the test Pi (AP6275P / BCM4362A2), root-caused by a kernel-config diff.

**Symptom (user-facing):** Bluetooth comes up working on **every boot** — pair, connect, stream
audio. But if you turn the Bluetooth **adapter** OFF (GNOME toggle, quick-settings, **Airplane Mode**,
or `rfkill`), it won't turn back **on** — the only recovery is a **reboot**. Any BT audio in progress
(e.g. a WiiM speaker) cuts out.

**Scope — it's the LOW-LATENCY KERNEL, not the chip:** the *identical* hardware + bring-up **recovers
fine** on a stock-preemption kernel (confirmed on a `PREEMPT_VOLUNTARY` / 300 Hz build — BT toggled
off→on came right back, reconnected to the WiiM). A full kernel-config diff between the recovering
build and the Pi Studio build shows the **only** difference is the low-latency settings
(`CONFIG_PREEMPT` full vs `VOLUNTARY`, `HZ=1000` vs `300`). So this is a Pi-Studio-specific tradeoff,
**not** a universal AP6275P/RK3588 bug — every stock-kernel board with this chip recovers normally.

**Rule for users:** *Once Bluetooth is on, leave the adapter ON.* Pair / connect freely — just don't
flip the master switch (or Airplane Mode) off. If it does go off → reboot.

**Mitigation SHIPPED ("Sandi's fix"):** the #1 way to trip this is an accidental mis-tap — the on/off
button sits right next to the device-select arrow. The ArmedOrange shell theme adds a **gap** between
them (`.quick-menu-toggle .quick-toggle-arrow { margin-left: 7px }` + both halves rounded) so a mis-
tap reaching for "connect a device" can't land on the kill-switch. Covers the common accident;
intentional off / Airplane Mode / CLI still need the "leave it on, reboot if off" rule. Sandi-approved.
In `assets/themes/ArmedOrange/gnome-shell/gnome-shell.css`.

**Cause (root, confirmed):** The AP6275P's Bluetooth is brought up by a userspace tool
(`brcm_patchram_plus`) that patches firmware over UART once at boot — a **timing-sensitive** sequence
(baud switching + 200 ms sleeps). The Rockchip vendor driver ties every "off" to the `BT_REG_ON` power
pin, so "off" power-cycles the chip and drops its firmware. On a stock kernel the live re-bring-up
reloads it fine; under Pi Studio's **full preemption + 1000 Hz** the re-init's timing is disrupted and
it fails to recover (boot works; live re-init doesn't). Confirmed: recovering vs bricking kernels
differ *only* in those low-latency settings.

**Fix path (post-launch) — keep low latency AND fix BT:** (a) run the patchram bring-up at realtime
priority (`chrt -f`) so full preemption can't interrupt it mid-sequence; (b) rebuild with
`CONFIG_PREEMPT_DYNAMIC=y` for a runtime `preempt=` knob (also lets us A/B-confirm); (c) ship a
stock-preempt kernel as a boot-menu fallback for users who prioritise reliable BT over min latency.
Full analysis: `pi-studio-bluetooth-serdev-fix.md`. Memory: [[bluetooth-ap6275p]].

---

## BT-2 — ✅ FIXED: Bluetooth failed to start at boot when WiFi was active (AP6275P combo-chip race)
**Status:** **FIXED** — root-caused, A/B-confirmed 4/4, **first-boot validated 2026-07-17** (WiFi live during BT bring-up → real MAC, `UP RUNNING`, zero `-49`). The fix ships in **builds after v1.2**; the public record is
[ubuntu-rockchip issue #1](https://github.com/defcom5-rockchip/ubuntu-rockchip/issues/1) + fix commit `814a50f`.

**What it was:** WiFi and Bluetooth share one physical chip (AP6275P / BCM4362A2). The stock bring-up script ran `rfkill unblock all` — waking WiFi *during* the BT firmware load — and fired patchram with no verify/retry. When WiFi won the race, `hci0` came up DOWN with a null MAC (`Opcode 0x1003 failed: -49`), Bluetooth greyed out, and only a lucky reboot recovered it.

**The fix (in `ap6275p-bluetooth.sh`):** `rfkill unblock bluetooth` — BT only, don't wake WiFi mid-load *(the actual fix)* — plus verify-the-MAC + one boot-time patchram retry as insurance.

**If you're on the v1.2 image (which predates the fix):** the old workaround still applies — on a wired connection, turn WiFi off and reboot; BT then comes up clean every boot. Or update the script by hand from the fix commit.

## BT-3 — Bluetooth *audio* limitations: WiFi coexistence + SBC codec (documented, not fixed)
**Status:** Known limits of the AP6275P radio + SBC; **by design we don't chase these** — the clean path is wired. Severity: low (BT audio remains usable; quality-critical listening should be wired anyway).

**BT audio to a speaker (e.g. a WiiM) can click/pop/stutter — two causes, both Bluetooth-only:**
1. **WiFi/BT coexistence** — the AP6275P shares one 2.4 GHz radio for WiFi *and* Bluetooth. **Turning WiFi on (even just scanning, not connected) wrecks BT A2DP audio.** Proven by a clean A-B-A: WiFi off = smooth tone, WiFi on = clicks/stutter, WiFi off = smooth again. **→ Use Ethernet and keep WiFi off for clean BT audio.**
2. **SBC codec on transients** — percussive/broadband material (drums) can click over BT even with WiFi off; sustained tones are fine. Inherent to lossy SBC on this chip. (An `sbc-xq` override was tested and is *not* a fix.)

**The clean path for bit-perfect audio to a WiiM is Ethernet, not Bluetooth** (see the hi-res how-to). For live Sonic Pi output destined for the speaker, render to a file and play it over Ethernet. *The audio engine itself is clean — the same material plays glass-smooth wired; every click above was the Bluetooth link.*

## DISP-1 — Windowed flicker / tearing at 4K@120
**Status:** Known limit (VOP2). **Workaround available.** Severity: low.
**Symptom:** At 4K@**120**, *windowed* GPU-composited content (browsers, apps) can flicker / mis-draw.
Fullscreen video is fine (direct scanout bypasses the compositor).
**Workaround:** run the desktop at **4K@60** (Settings → Displays → Refresh Rate); use 4K@120 for
fullscreen video/motion. It's the VOP2 compositor frame-timing ceiling, not a decode/browser bug.

## BROW-1 — Vivaldi must run with `--disable-gpu`; Chromium is the HW-accel browser
**Status:** Known (immature Panfrost). **Fixed via launcher flag.** Severity: low.
**Symptom:** Vivaldi (upstream Chromium, no rkmpp glue) flickers / drops tiles even at 4K@60 because
it runs under XWayland on this board's immature GPU stack, and it software-decodes video.
**Fix shipped:** launch Vivaldi with `--disable-gpu` (clean software render). For hardware video
decode + clean rendering, use **Chromium** (the rkmpp build, native Wayland) — the designated primary
browser. See `HANDOFF-image-build.md` §8 and [[chromium-hwdecode-pistudio]].

## STREAM-1 — Streaming DRM works, but Netflix (in-browser) does not
**Status:** Known platform limitation (arm64 Widevine ceiling). Not fixable in-browser. Severity: low.
**What works:** Widevine DRM plays fine — **Amazon Prime Video, Spotify, DRM YouTube, Disney+** —
after DRM is enabled (an on-demand helper; not bundled, for licensing reasons).
**What doesn't:** **Netflix in a browser** → `M7701-1003` / `E100` ("Widevine outdated") in *both*
Chromium and Vivaldi.
**Why:** Netflix demands a newer Widevine than the **only CDM that exists for arm64 Linux**
(`4.10.2662.3`, extracted from ChromeOS — deprecated for arm64 at Chrome 130, so it's a *permanent*
ceiling). Google ships no arm64 Widevine and no arm64 auto-updater. This hits **every** arm64 Linux
browser, not just Pi Studio.
**Workaround:** Waydroid + the Netflix Android app (its own L3 Widevine), or an external stick (Fire
TV / Chromecast). **Marketing/docs must say "Widevine DRM," never "Netflix."**
See `widevine-drm-howto.md`, memory [[widevine-vivaldi-arm64]].

## MAINT-1 — No automatic updates (by design) → you patch on your own cadence
**Status:** Deliberate design choice, not a bug. Severity: informational (but *must* be disclosed).
**What it means:** Pi Studio ships with **no auto-updates** — nothing patches itself behind your back
(an audio machine shouldn't churn CPU or reboot mid-session for an update; *you* decide when). The
flip side is real: **security patching is now a manual act** — nobody does it for you. Run it on your
own schedule:
```bash
sudo apt update && sudo apt upgrade      # userspace security fixes (curl, openssl, etc.)
```
**Kernel:** the custom 4K@120 low-latency kernel is `apt-mark hold`'d, so `apt upgrade` will NOT touch
it (correct — that's what protects your kernel). Kernel security therefore does **not** arrive via
`apt` — it comes with **Pi Studio *image* releases** (a periodic rebase onto the latest `6.1.y` stable
point release, which sweeps in upstream CVE fixes). So: **patch userspace yourself; watch for image
updates for the kernel.** This is the honest cost of a hand-tuned kernel — worth knowing, not hiding.

---
*Add new verified-on-hardware limitations here as they're found. Honesty > hiding.*

— Pi Claude
