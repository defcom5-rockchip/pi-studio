# External Storage — Pi Studio

Which drive to buy, which port to plug it into, and why the fastest SSD on the shelf is
the wrong purchase for this board.

*Written August 2026. The technical limits are permanent; the pricing section is a snapshot —
see [Market conditions](#market-conditions-august-2026).*

---

## The short version

| | |
|---|---|
| **Audio interface** | USB **2.0** port |
| **Storage** | USB **3.0** port |
| **Buy** | 1–2 TB **SATA-based** SSD, or a bare 2.5" SATA SSD in a **UASP** enclosure |
| **Don't buy** | NVMe-in-an-enclosure — you'll pay double for speed this board can't use |
| **Archive** | A spinning drive is still a third of the price per TB |

---

## Put the audio interface on USB 2.0. Really.

This feels wrong and it isn't.

**Class-compliant audio interfaces are USB 2.0 devices**, including expensive ones. The USB Audio
Class 2 specification runs at high speed — 480 Mbit/s — and that is what your interface negotiates
no matter which port it's in. An RME Babyface Pro plugged into a blue USB 3.0 port comes up at
*USB2 480M*; the SuperSpeed lanes in that connector sit unused.

So putting the interface in a USB 2.0 port costs you nothing, and it frees the 3.0 port for the one
device that can actually use it.

It is also the better arrangement for audio. Keeping bulk storage transfers off the same path as an
isochronous audio stream removes a classic cause of xruns and dropouts. Sustained writes to a drive
sharing a controller with your interface is exactly the kind of contention that produces clicks
under load.

For scale: a 24-bit/192 kHz stereo stream is about **1.1 MB/s**. Thirty-two tracks at 192 kHz is
under **40 MB/s**. Audio is not a bandwidth problem.

> **Note:** with the u-boot fix in Pi Desktop v1.0 and current Pi Studio images, you can leave a
> bus-powered interface connected at power-on. Earlier images could hang in the bootloader before
> reaching the kernel.

---

## The port is the ceiling — not the drive

The RK3588S exposes **USB 3.0 at 5 Gbit/s**. After protocol overhead, real-world throughput lands
around **400 MB/s**. That number decides your purchase:

| Drive type | Native speed | What you get here |
|---|---|---|
| SATA SSD (external or in enclosure) | ~550 MB/s | **~400 MB/s — port saturated** |
| NVMe SSD in a USB enclosure | 2,000–7,000 MB/s | **~400 MB/s — identical** |
| Spinning HDD | 150–200 MB/s | 150–200 MB/s |

A SATA SSD already exceeds what the port can carry. Every pound spent above that tier buys
bandwidth the board physically cannot deliver.

**There is no NVMe option on the Orange Pi 5B.** The board exposes a single PCIe 2.0 x1 controller,
and on the 5B that lane serves the onboard WiFi rather than an M.2 slot. USB 3.0 *is* the fast
storage path on this hardware — another reason not to waste that port on a 480 Mbit device.

---

## UASP matters more than the drive does

**USB Attached SCSI Protocol** is a property of the *enclosure*, not the drive — and it decides how
much of your SSD actually reaches the board.

Every enclosure contains a USB-to-SATA bridge chip, and that chip speaks one of two protocols. **BOT**
(Bulk-Only Transport) is the old one, built for USB flash drives: one command at a time, wait for
completion, send the next. **UASP** queues multiple commands, supports NCQ, and lets the drive
reorder work.

Expect roughly **250–350 MB/s on BOT** against **400+ MB/s on UASP** for sequential transfers. The
larger gap is in random I/O, queue depth and CPU overhead — which is what you feel when a DAW is
streaming samples while something else writes in the background.

**Confirming before you buy:** good enclosures advertise "UASP" outright. If the listing doesn't
mention it, look for the bridge chipset — **ASMedia ASM1153/ASM1351**, **JMicron JMS578/JMS583** or
**Realtek RTL9210** all support it. An enclosure that names no chipset and never says UASP is the
one to skip.

Verify on the board:

```sh
lsusb -t
```

Look for `5000M` (not `480M`) and a `uas` driver on your drive's line. If you see `usb-storage`
instead of `uas`, you're on the slow path.

---

## Power

The 5 V budget is real, and it is the same limit that used to stop this board booting with a
bus-powered audio interface attached.

- Prefer a **low-draw** drive, or one with its own power supply for larger 3.5" units.
- Use a **short, thick cable**. Voltage drop over thin conductors causes dropouts, random
  disconnects, and enumeration failures that look exactly like driver bugs.
- Cheap printer/scanner cables are built for devices that draw almost nothing. They work, but they
  are the first thing to swap if a bus-powered device misbehaves.

Check whether the port is complaining:

```sh
dmesg | grep -iE 'over-current|reset high-speed'
```

Clean output means the cable and power are fine.

---

## Capacity: 2 TB is the sweet spot

**1 TB** fills faster than expected once sample libraries, project files and renders accumulate.
**2 TB** is the practical working size for most people. Beyond that you're paying a premium for
capacity that would serve you better as a cheap archive drive.

Split the job if your library is large:

- **SSD** — active projects, the sample libraries you stream from, anything the DAW touches live.
- **HDD** — finished projects, backups, archives. Around a third of the price per TB, and USB 3.0
  saturates a spinning drive easily.

---

## Market conditions (August 2026)

Storage is expensive right now and this is worth knowing before you shop.

NAND flash contract prices carry roughly a **4.2–4.5× multiplier** compared with late 2025 —
+33–38% in Q4 2025, +55–60% in Q1 2026, +70–75% in Q2 2026 — driven by a global NAND shortage and
demand from the AI sector. In practice a 2 TB NVMe that cost around $175 in November 2025 was near
$379 by March 2026, and 1 TB M.2 drives moved from roughly $65–70 to $130 and up. Supply is not
expected to steady for a year or more.

Two consequences:

**The price spike hits hardest on exactly the drives you shouldn't buy.** The NVMe segment is where
the increases are steepest, and it's the segment that gains you nothing on this board. SATA drives
use older NAND that the AI buildout is competing for less aggressively, and they saturate your port
anyway.

**Waiting is not a strategy.** With supply unlikely to normalise soon, buy the capacity you need in
the value tier now rather than holding out for a correction that forecasters don't expect this year.

A bare 2.5" SATA SSD plus a UASP enclosure is frequently cheaper per terabyte than a sealed portable
drive, and you keep the enclosure when you upgrade.

---

## Measuring what you actually got

With the drive mounted:

```sh
# confirm it's on a SuperSpeed port with the fast driver
lsusb -t

# sequential read
sudo hdparm -Tt /dev/sda

# sequential write (adjust the mount path)
dd if=/dev/zero of=/mnt/ssd/testfile bs=1M count=2048 oflag=direct status=progress
rm /mnt/ssd/testfile
```

**Expected:** 350–420 MB/s read and write on a healthy SATA SSD over UASP.

**If you see 250–350 MB/s:** likely BOT instead of UASP. Check `lsusb -t` for `usb-storage` rather
than `uas` — that's the enclosure, not the drive.

**If you see ~35 MB/s:** you've dropped to USB 2.0 entirely. That's a cable, port or enclosure
fault, not a protocol one.

**If `lsusb -t` shows 480M:** it fell back to USB 2.0. Usually the cable or the enclosure. Try a
different cable before blaming anything else.

---

*Part of [Pi Studio](README.md). See also [KNOWN-ISSUES](KNOWN-ISSUES.md) and
[EMMC-INSTALL](EMMC-INSTALL.md).*
