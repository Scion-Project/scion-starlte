<!-- SPDX-License-Identifier: MIT-0 -->
# The Galaxy S9's hardware: what Scion drives, and what it leaves alone (17 November 2026)

A map, not a plan of record. It is made from Samsung's device tree for this phone
(`exynos9810-starlte_eur_open_26.dts`: 434 nodes with a `compatible`, 329 of them enabled, 183 kinds), read as data,
set against the drivers in scion-kernel and the states in `device.conf`. "Not driven" means no code of Scion's touches
the block; it does not mean the block cannot be made to work, and nothing below has been tried unless it says so.
Addresses and numbers are the device tree's.

## Driven today

| Block | By | Note |
|---|---|---|
| Eight cores, GIC-400, timers | the kernel, `hakios.exynos_m3.c` | all eight run; **idle is WFI only** (see "Power") |
| Clusters' and GPU's clocks | `hakios.exynos_cpufreq.c`, `hakios.scion_gov.c`, `hakios.mali.c` through ACPM | the cores and the GPU both slow with heat (the GPU from 85 C: on 16.5 by its governor, on iOS 27 since 17 November); a run ends at 110 C |
| UFS | `hakios.ufs.c`, `hakios.ufshc.c` | no inline encryption (FMP), no RPMB |
| USB | `hakios.dwc3.c`, `hakios.exynos_usb.c` | a device only, on the 2.0 PHY; no host mode, no SuperSpeed |
| Display | `hakios.exynos_dpu.c` (DECON, DPP, DSIM), `hakios.s6e3ha8.c` | one DECON of two; no DisplayPort |
| GPU | `hakios.mali.c` (Mali-G72 MP18) | one job slot at a time, finished jobs found by polling |
| Video | `hakios.exynos_mfc.c` | decode only |
| Buses | `hakios.exynos_hsi2c.c`, `hakios.exynos_spi.c`, ACPM | |
| Wi-Fi, Bluetooth | `hakios.exynos_pcie.c` + brcmfmac, `hakios.bcm4361_bt.c` | |
| Audio | `hakios.exynos_abox.c`, `hakios.max98512.c`, `hakios.cs47l93.c` | ABOX with its own processor off: one playback path, one microphone |
| Modem | `hakios.shannon.c` | data; no calls, no SMS |
| Sensors | `hakios.ssp.c` (the hub in the BCM4775) | six sensors |
| Cameras | `hakios.exynos_camera*.c`, `hakios.ak737x.c`, `hakios.exynos_iris.c` | sensors and CSIS only: frames are raw, demosaiced in software |
| Touch, keys, haptics, torch, battery, RTC, heat | `hakios.s6sy761.c`, `hakios.exynos_keys.c`, `hakios.haptic.c`, `hakios.s2mpb02_torch.c`, `hakios.max77705.c`, `hakios.s2mps18_rtc.c`, `hakios.exynos_tmu.c` | charger and gauge read only |

## Not driven

### Power: the largest gap

| What the device tree has | State |
|---|---|
| **Core power-down in idle ("c2")**: PSCI `CPU_SUSPEND` (function 0xC4000001) with parameter 0x10000; entry/exit 70/160 us on the A55s, 235/220 us on the M3s, worth it from 2 ms and 3.5 ms of idle | not used: an idle core sits in WFI, powered |
| **Cluster power-down and system idle clock-down** (`exynos-powermode`: `cpd`, `sicd`, both enabled, 10 ms residency) | not used |
| **Suspend** (`exynos-pm`: mode index 8, PSCI index 133; wake-ups from the keys, the RTC, the modem's mailbox, USB) | not used: the phone never sleeps (what a lock does to the panel has not been checked for this page) |
| Twelve power domains (`pd-aud`, `-dpu`, `-g2d`, `-isphq`, `-isplp`, `-isppre`, `-iva`, `-mfc`, `-vts`, three for the display controller) | switched on where a driver needs one; none is switched off again when idle |
| Bus and memory frequency scaling (`devfreq`: MIF, INT, INTCAM, DISP, CAM, AUD, IVA, SCORE) | not used: the rates are as the boot loader left them |
| CPU hot-plug, the over-current protector of the big cores (`exynos-ocp`) | not used |
| Wireless charging (IDT, `mfc-charger`), charging control | not used; the charger is read only by decision |

### Media and vision

| Block | State |
|---|---|
| **The camera's hardware pipeline** (FIMC-IS: 3AA, ISP, scaler, phase-detect `pdp`, distortion correction `gdc`) with Samsung's library in its reserved memory | not used. It is why the cameras give raw frames. The library is firmware of Samsung's |
| **Video encode** (the same MFC) | not used: nothing can record video in hardware |
| IVA and SCore, the two vision accelerators | not used |
| G2D, the scaler (`scaler@17710000`), JPEG (`smfc`), ASTC | not used; iOS has no consumer for them as they are |
| The second DECON and **DisplayPort over USB-C** (controller, PHY, its audio) | not used |

### Connectivity and storage

| Block | State |
|---|---|
| **GNSS** (the BCM4775's own function) | researched, no fix yet: `docs/gnss.md` |
| NFC (`sec-nfc`), the secure element (`ese_p3`), MST | not used |
| **microSD** (`dwmmc2`) | a design exists for the iOS 27 tree (scion-kernel `docs/ios27/XNU-MICROSD.md`); nothing here |
| USB host mode, SuperSpeed | not used |

### Sensors and input

| Block | State |
|---|---|
| Heart-rate sensor (`hrmsensor@57`), grip sensor (`sx9320`), the two Hall sensors | not used |
| Fingerprint reader (`etspi`) | not used; it answers only to Samsung's secure world |
| Iris camera | the sensor answers, no frame reaches memory |

### Audio

| Block | State |
|---|---|
| ABOX's own processor, its effects, seven more playback and four more capture paths, the other three UAIFs | not used |
| Voice trigger (`vts`), the always-on microphone path | not used |
| Headphone jack | written behind `cs47l93_hp=1`, not yet tried |
| Earpiece | not used |

### Left alone on purpose

The secure world (`exynos-tee`: probed only, behind `tee_probe=`), RPMB, key storage; the watchdogs (`wdt=-1`); the
IOMMUs (sixteen `sysmmu`: the drivers use fixed memory instead); CoreSight and Samsung's debug snapshot.

## What would be worth the most, in order

1. **Core power-down in idle.** Every number it needs is in the device tree, the call is PSCI's, and it is where the
   battery and the heat go. It needs a phone for every step: a core that does not come back is a hang.
2. **Suspend with the screen off**, after 1.
3. **The GPU's job interrupt and its second job slot** (scion-kernel `docs/ios27/LIQUID-GLASS.md`: what a frame waits for).
4. ~~A limit on the GPU's clock with heat~~: 16.5 had it (this page's first version was wrong to say otherwise); the
   iOS 27 tree has it, and a stop at 110 C, since 17 November (Scion-Project/xnu #27). Not yet run on a phone there.
5. **Video encode**, then the camera's pipeline, for a Camera app that records.
6. DisplayPort, microSD, GNSS: each a project of its own, none blocking daily use.
