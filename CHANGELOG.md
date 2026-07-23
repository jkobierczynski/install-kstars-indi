# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

Nothing yet.

---

## [0.6.0] - 2026-07-24

Device firmware, PHD2 and plate-solving indexes.

### Added

- `fxload` to the Ubuntu package set. FX2-based QHY cameras (QHY5 and
  relatives) upload firmware at plug time and will not enumerate without it.
  Installation failure is reported as a warning rather than a fatal error,
  since the usual cause is `universe` not being enabled.
- Corrected QHY udev rules written to `/etc/udev/rules.d/85-qhyccd.rules`.
  udev reads `/etc` before `/lib`, and a same-named file there fully shadows
  the packaged copy, so the fix survives `indi-qhy` upgrades.
- Source build for PHD2 from `OpenPHDGuiding/phd2`, used only when no package
  exists for the target release. Controlled by `phd2_build_from_source`,
  `phd2_repo` and `phd2_version`.
- Astrometry.net solver and index-file packages, behind `install_astrometry`.
  Index selection is exposed as `astrometry_index_packages` and defaults to
  the 4208–4219 series (roughly 2–30 arcmin fields).
- Post-install verification that QHY firmware files are present in
  `/lib/firmware/qhy`, with a summary of how many were found.

### Fixed

- Doubled firmware path `/lib/firmware/qhy/qhy/` in the shipped
  `85-qhyccd.rules`, which prevented firmware upload for QHY devices.
  The substitution is anchored on the trailing slash. The unanchored form
  is **not** idempotent: on an already-correct file it rewrites
  `/lib/firmware/qhy/qhy5loader.hex` to `/lib/firmware/qhy5loader.hex`,
  breaking re-runs and any system where the packaging has since been fixed.

### Notes

- PHD2 build dependencies are probed rather than pinned. wxWidgets 3.0, 3.2
  and 3.4 coexist across releases alongside the `t64` ABI rename, so a fixed
  list breaks on upgrade.
- StellarSolver (inside Ekos) and astrometry.net consume the same index
  format. The indexes are required regardless of which solver is used; the
  solver binaries are the optional part.

---

## [0.5.0] - 2026-07-24

Ubuntu derivative support and headless installs.

### Added

- `install_kstars` toggle. Setting it to `false` installs INDI, the driver
  set and the web manager without the GUI, for the server-side host in a
  split observatory.
- `breeze-icon-theme` alongside KStars. No-op on Kubuntu; prevents missing
  toolbar icons on GNOME-based Ubuntu.

### Fixed

- `ppa_suite` is now derived from `UBUNTU_CODENAME` in `/etc/os-release`
  rather than `ansible_distribution_release`. Derivatives such as Linux Mint
  report their own codename (`xia`, `wilma`), which produces a 404 against
  `ppa.launchpadcontent.net` that is indistinguishable from "the PPA has not
  built for this release yet".

### Changed

- The `kstars-bleeding` availability assertion only runs when
  `install_kstars` is true, so headless runs are no longer failed by the
  absence of a package they did not ask for.

### Notes

- Ubuntu flavours (Kubuntu, Xubuntu, Lubuntu, MATE, Budgie) all report
  `ansible_distribution == 'Ubuntu'` and need no special handling. The
  `'Kubuntu'` entry in the distribution match list is dead code, retained
  only for readability.

---

## [0.4.0] - 2026-07-23

Package conflict resolution.

### Fixed

- `indi-full` and `indi-3rdparty-drivers` declare `Conflicts:` against each
  other. The availability probe introduced in 0.3.0 selected both, producing
  `E: Unable to satisfy dependencies`. Exactly one driver bundle is now
  chosen, preferring `indi-3rdparty-drivers` where it exists.
- On Ubuntu 26.04 `indi-full` is stale (2.2.0, April) against `libindi`
  2.2.2 (June) and depends on packages that are no longer installable:
  `indi-qsi`, `indi-atik`, `indi-toupbase`, `indi-svbony`, `indi-duino`,
  `indi-astroasis`, `indi-rolloffino`, `indi-avalonud`. This confirms
  empirically that `indi-3rdparty-drivers` supersedes it on 26.04.

### Changed

- `libindi-dev` moved behind `install_indi_dev`, default `false`. Headers
  are only needed to compile against INDI.

### Removed

- `indi-bin` and `libindi1` from the explicit package list. Both arrive
  transitively; naming them only gave apt more versions to pin and conflict
  on.

### Added

- Debug output of the resolved package set before installation, so a bad
  selection is visible before apt is invoked.

---

## [0.3.0] - 2026-07-23

PPA identity correction.

### Fixed

- `ppa:mutlaqja/kstars` does not exist and returned 404 from the Launchpad
  API. There is a single PPA, `ppa:mutlaqja/ppa`, carrying both the INDI
  library and `kstars-bleeding`.
- Nightly PPA name corrected to `ppa:mutlaqja/indinightly` in the README.

### Changed

- Package names are probed with `apt-cache show` rather than hardcoded.
  Upstream documentation and the search-indexed revision of the same page
  disagreed on whether the driver bundle is `indi-full` or
  `indi-3rdparty-drivers`; probing avoids betting on either.

---

## [0.2.0] - 2026-07-22

Keyserver-free repository setup.

### Fixed

- `apt_repository` with `ppa:` shorthand fails with
  `Unable to get required signing key` when `hkp://keyserver.ubuntu.com:80`
  is unreachable or `dirmngr` is absent. gpg exits 0 having exported nothing,
  so the failure is silent until apt reports a missing key.
- `deb822_repository` requires `python3-debian` on the target and aborts at
  import time without it. Replaced with a direct `.sources` file write.
- The missing-suite check was unreachable. `failed_when` overwrites the
  registered failure flag, so the subsequent `when: ... is failed` could
  never evaluate true and the intended diagnostic never printed.
- YAML parse error caused by an unquoted colon inside a task name.

### Added

- `HEAD` probe of `dists/<suite>/Release` before `apt update`, so a PPA with
  no build for the running release fails with an actionable message instead
  of an opaque apt 404.
- Removal of stale `.list` files left behind by earlier `apt_repository`
  runs, which otherwise keep triggering missing-key errors after the
  `.sources` file is in place.

### Changed

- Signing keys are fetched over HTTPS from the Launchpad API and
  `keyserver.ubuntu.com/pks/lookup`, then written to `/etc/apt/keyrings`
  and referenced with `Signed-By`. No gpg keyserver traffic is involved.
- Downloaded keys are validated with `gpg --show-keys` before use, so an
  empty or truncated download fails at fetch time.

---

## [0.1.0] - 2026-07-22

Initial playbook.

### Added

- Ubuntu/Kubuntu installation path via the official INDI PPAs.
- Debian installation path with two strategies, selected by
  `debian_install_method`:
  - `source` (default): builds INDI core, `indi-3rdparty` libraries and
    drivers, StellarSolver and KStars from the newest upstream release tags.
  - `distro`: installs Debian's own packages.
- Release tags resolved at run time via `git ls-remote --tags` and `sort -V`,
  with per-project pinning through `indi_version`, `stellarsolver_version`
  and `kstars_version`.
- Build dependencies discovered via `apt-get build-dep` plus an
  `apt-cache`-filtered candidate list, so the Qt5/KF5 to Qt6/KF6 migration
  does not break the playbook across releases.
- `deb-src` enablement for both deb822 (`*.sources`) and legacy
  `sources.list` formats.
- udev rules installed under `/usr/local` symlinked into `/etc/udev/rules.d`,
  which udev does not otherwise scan. Without this, cameras are only
  accessible as root.
- Device access group membership (`dialout`, `plugdev`, `video`, `tty`) for
  the configured `astro_user`.
- Optional `usbcore.usbfs_memory_mb` tuning for large-sensor cameras, behind
  `tune_usbfs_memory`.
- Optional PHD2 and INDI Web Manager installation.
- Example inventory covering a local run, a desktop client and a
  mount-side host.

---

[Unreleased]: https://github.com/USER/REPO/compare/v0.6.0...HEAD
[0.6.0]: https://github.com/USER/REPO/compare/v0.5.0...v0.6.0
[0.5.0]: https://github.com/USER/REPO/compare/v0.4.0...v0.5.0
[0.4.0]: https://github.com/USER/REPO/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/USER/REPO/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/USER/REPO/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/USER/REPO/releases/tag/v0.1.0
