<!-- SPDX-License-Identifier: MIT-0 -->
# The Exynos 9810's video codec (MFC) firmware (prebuilt, Samsung's)

NOTICE. Samsung's MFC firmware image, `/vendor/firmware/mfc_fw.bin` of the phone's Android: the code the SoC's hardware video
codec runs. It belongs to its owners and is not licensed by this repository (LICENSE, item 5).

**Not in this repository.** Only this README and `SHA256SUMS` are tracked; the file is fetched, and checked against
`SHA256SUMS`, by `tools/fetch-firmware.sh` (from your own copy with `--from DIR`, else from the pinned public community
vendor tree it names; `--check` only verifies). `userland/build.sh` (with `DEVICE=starlte`, the default for a kernel
configured for the S9) runs it and stops with what to do if the file cannot be had.

Source: the LineageOS 24 vendor tree for the Exynos 9810, ExyHyperBrick/proprietary_vendor_samsung_exynos9810-common at
2b130daba60b, `proprietary/vendor/firmware/mfc_fw.bin` (682192 bytes): the same bytes as the S9+'s
(devices/star2lte/firmware/media), the S9's and the Note9's.

`userland/build.sh` puts it into `fw/` of the 16.5 image, where the kernel's `drivers/media/mfc/hakios.exynos_mfc.c`
(`mfc=1..5`, `mfc=5` a default of configs/starlte.config) takes it; only without it there does the driver look in the
phone's own VENDOR partition at run time, as it did before this file was named here (until 17 November 2026 every S9 image
was built so, and a phone whose VENDOR has another ROM, or none, had no video decoder). The iOS 27 kernel has no MFC driver:
its root build checks this file and does not use it.

| file | what it is | sha256 |
|---|---|---|
| `mfc_fw.bin` | the MFC's firmware (682192 bytes) | `378f85ce94f8a3f6700c0bf17f2cd0348809728e26d678cf06204fc528f368d4` |
