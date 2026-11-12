# Licensing

This file is a plain-English map of who may use what under which licence. It does not replace the licence files.

The repository-level `LICENSE` is MIT with a by-directory description, and is not changed; `NOTICE` lists the third-party parts and is not changed.

## Original work: MIT-0

Files that the Scion project wrote entirely from scratch (every commit by the project's authors, no third-party notice, nothing ported or derived from another codebase) are licensed **MIT-0** (MIT No Attribution, SPDX `MIT-0`; text in `LICENSES/MIT-0.txt`). Each such file carries `SPDX-License-Identifier: MIT-0` in its first lines, except the files listed in the table below, which cannot hold a comment.

MIT-0 paths (top-level location, number of files):

- `.github`: 1 file
- `(top level)`: 9 files
- `configs`: 2 files
- `docs`: 2 files
- `firmware`: 4 files
- `tools`: 1 file
- `uefi`: 1 file

The full per-file list is in the pull request that introduced this file, and every file's own first lines are authoritative.

## Files that keep their existing licence

- Anything not listed above, and any file that has no explicit `SPDX-License-Identifier: MIT-0` tag, stays under the repository's existing licence, as before.
- `uefi/files/*`: changes to Project Silicium's Mu-Silicium, BSD-2-Clause-Patent.
- `prebuilt/*`: Mu-Silicium builds (BSD-2-Clause, BSD-2-Clause-Patent, GPL-2.0 for the Carbon Hypervisor) and AOSP's `mkbootimg.py` (Apache-2.0).
- `firmware/*/SHA256SUMS`: checksum lists of Samsung's and Broadcom's firmware, which is not in this repository and not licensed by it.
- `porting.toml`, `board.toml`, `SENSORS.md`: see the PR.

