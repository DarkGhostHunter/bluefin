# NVIDIA 580.x driver and kernel target for a ublue-os/bluefin fork (as of 2026-11-15)

Investigator: GLM-5.3-Flash subagent following the evidence-investigator skill; parent verified decisive facts (see Addendum). Working files: none (read-only subagent); tree kept in reasoning.

## 1. Abstract
- **What was asked:** For a Bluefin fork built today: latest NVIDIA 580.x driver, the newest kernel it supports (7.x preferred), and the Fedora + ublue packaging pieces that deliver that exact pair.
- **Results:** Driver 580.196.07. Kernel target 7.3.x (7.x is proven compatible; no 6.x fallback needed). Fedora 45 ships 7.3.x kernels. The ublue piece is the `akmods-nvidia-lts` image, tag `main-45-7.3.3-300.fc45`, built from the negativo17 `fedora-nvidia-580` repository, which currently serves `nvidia-driver 580.196.07-1.fc45`.
- **What can be done further:** Watch for an akmods rebuild tagged for kernel 7.3.4 and for a 580.x release newer than 580.196.07.

**ELI10:** The newest 580-series NVIDIA driver is 580.196.07. Linux kernel 7.3 is the current stable kernel and this driver builds on it; ublue has already built and published the 580 driver package for kernel 7.3.3 on Fedora 45. So build your fork on Fedora 45 with kernel 7.3.3 and use ublue's `akmods-nvidia-lts:main-45-7.3.3-300.fc45` image.

## 2. Prompt description
Unattended run. Assumptions:
1. "Latest 580.x" means the newest release in the NVIDIA 580 production/LTS branch, not the newest NVIDIA branch overall (RPM Fusion's current `akmod-nvidia` is 620.70.06, out of scope).
2. The fork is x86_64 and uses ublue packaging conventions (ghcr.io ublue-os images).
3. "Kernel 7.x preferred" means the newest kernel.org stable series that the driver demonstrably builds and ships against.

## 3. Process
Leaves investigated in order: kernel.org release table; NVIDIA download archive probes plus `open-gpu-kernel-modules` tags (release dates); compatibility evidence (driver README, GitHub issues for kernel 7.x, and decisive: an existing ublue build artifact pairing 580 with a 7.3 kernel); Fedora release state (Local machine rpm label + search confirmation); ublue-os/akmods version-selection mechanics (workflows, build scripts, GHCR tag probes); bluefin stream definitions (workflows and Justfile). The compatibility leaf was settled by an artifact rather than prose: GHCR already contains the built 580-vs-7.3.3 pair, which is stronger than a forum claim. The skill's `scripts/check_report.py` does not exist; structure was self-checked against `references/report-template.md`.

## 4. Findings

### F1. Latest NVIDIA 580.x release is 580.196.07
**Assumptions / corrections:** None.

> "# NVIDIA Linux Open GPU Kernel Module Source ... version 580.196.07." [^1]

The `.run` installer exists on NVIDIA's download server: probing `https://us.download.nvidia.com/XFree86/Linux-x86_64/580.196.07/NVIDIA-Linux-x86_64-580.196.07.run` returned HTTP 200, and the tag date is 2026-10-21. [^2] Older 580.x tags (580.178.04 dated 2026-08-03, 580.173.02 dated 2026-06-25) are strictly older, and no 580.x tag above 580.196.07 exists in the repo's tag list. [^2]

**Interpretation:** 580.196.07 is the newest 580.x release as of 2026-11-15.
**Affects:** All later leaves use 580.196.07 as the driver version.

### F2. Newest stable kernel is 7.3.4; newest maintained 6.x is 6.18.63
**Assumptions / corrections:** The Local data point (this machine runs `7.3.3-300.fc45.x86_64`) matches the Official kernel.org table.

> kernel.org lists "stable: **7.3.4** 2026-11-10" and "longterm: **6.18.63** 2026-11-10"; mainline is "7.4-rc2". [^3]

**Interpretation:** The 7.x series exists and 7.3 is the newest stable series. 6.18.63 is the newest maintained 6.x longterm.
**Affects:** F3 (kernel choice) and F5 (Fedora mapping).

### F3. Verdict on kernel 7.x: proven compatible — no 6.x fallback
**Assumptions / corrections:** None.

The driver README sets no upper kernel bound:

> "The NVIDIA open kernel modules support the same range of Linux kernel versions that are supported with the proprietary NVIDIA kernel modules. This is currently Linux kernel 4.15 or newer." [^4]

Decisive artifact: ublue has already built and published the 580 driver stack against a 7.3 kernel. GHCR manifests exist for `ublue-os/akmods-nvidia-lts:main-45-7.3.3-300.fc45` and `ublue-os/akmods-nvidia-open:main-45-7.3.3-300.fc45` (HTTP 200), while `main-45-7.3.4-300.fc45` does not yet exist (404). [^5] The `nvidia-lts` target builds the akmod in CI, and a failed akmod build exits the workflow ("cat ...failed.log && exit 1"), so a published image proves the build succeeded on that kernel. [^6]

Third-party noise exists ("Driver 580 does not compile/build under kernel 7.0" reports on NVIDIA forums, and kernel-7.3 runtime issues #1507/#1512 filed against driver 615.78.08), but those are older driver versions or non-build defects; they do not prove 580.196.07 is incompatible with 7.3. [^7] [^8] "No evidence" is not incompatibility, and here positive build evidence exists.

**Interpretation:** Target kernel 7.3.x. The exact published pair is 7.3.3-300.fc45; 7.3.4 (kernel.org, 2026-11-10) has no published akmods artifact yet.
**Affects:** F5, F6, and the verdict.
**Conflict:** Third-party reports of 580 build trouble on early 7.x kernels versus a published build artifact for 580.196.07 on 7.3.3. The artifact (from the actual packaging repo, closer to Official) prevails; earlier-kernel failures were fixed before 580.196.07.

### F4. Fedora state: 45 is current stable and ships 7.3.x
**Assumptions / corrections:** None.

> Fedora Linux 45 was announced as stable on November 3, 2026; 44 is the previous supported release. [^9]

Local: the machine reports `Linux version 7.3.3-300.fc45.x86_64` (OS Version in this session's environment), consistent with Fedora 45 carrying the kernel.org 7.3 stable series. [^10]

**Interpretation:** Fedora 45 is the base that ships kernel 7.3.x; ublue akmods builds for `version: 45` confirm packaging support. [^6]
**Affects:** F6.

### F5. How ublue-os/akmods selects the NVIDIA driver version
**Assumptions / corrections:** None.

Driver selection is repository-driven, not hard-coded. The nvidia-lts akmods target consumes negativo17's 580 repository:

> "curl -L https://negativo17.org/repos/fedora-nvidia-580.repo ... sed -i 's/^\[fedora-nvidia-580/[fedora-nvidia-lts/'" [^11]

That repo file points to `baseurl=https://negativo17.org/repos/nvidia-580/fedora-$releasever/$basearch/`. [^11] The build script installs the newest `akmod-nvidia` in that repo:

> "DRIVER_VERSION=$(dnf info akmod-nvidia | grep -E '^Version|^Release' | awk '{print $3}' ...)" — with an optional `NVIDIA_AKMOD_EVR` pin installing "exactly the EVR recorded in the image fingerprint". [^6]

The negativo17 `nvidia-580` repo for Fedora 45 currently serves exactly the target driver:

> `<name>nvidia-driver</name> ... <version epoch="3" ver="580.196.07" rel="1.fc45"/>` and `<name>akmod-nvidia</name> ... <version epoch="3" ver="580.196.07" rel="1.fc45"/>`. [^12]

Image tags are `<kernel_flavor>-<fedora_version>-<kernel_version>`, e.g. the workflow builds `akmods_target: nvidia-lts` with `kernel_flavor: main, version: 45`, published as `ghcr.io/ublue-os/akmods-nvidia-lts:main-45-<kernel>`. [^6] [^5] There is no workflow-dispatch input for a driver version; the only dispatch input is `force`. [^6] By contrast, RPM Fusion's main `akmod-nvidia` is 620.70.06-2.fc45, which is what the `nvidia-open` target (bluefin's default NVIDIA flavor) picks up. [^13]

**Interpretation:** To pin 580.x you use the `nvidia-lts` akmods target/image, not `nvidia-open`; pinning to an exact 580 EVR is possible via `NVIDIA_AKMOD_EVR`.
**Affects:** F6 and the verdict.

### F6. Bluefin streams and how they consume akmods
**Assumptions / corrections:** GTS was not found as a current stream name in the workflows; current streams are `stable` and `latest`.

Bluefin pulls prebuilt kmod RPMs by tag and only builds `nvidia-open` images:

> "skopeo copy ... docker://ghcr.io/ublue-os/akmods-nvidia-open:"${AKMODS_FLAVOR}"-"$(rpm -E %fedora)"-"${KERNEL}"" and "if [[ "${IMAGE_NAME}" =~ nvidia ]]; then ... akmods-nvidia-open". [^14]

Stream-to-akmods-flavor mapping in the Justfile:

> "if [[ "${flavor}" =~ hwe ]]; then akmods_flavor="bazzite"; elif [[ "${tag}" =~ stable ]]; then akmods_flavor="coreos-stable"; else akmods_flavor="main"; fi" [^15]

Stable builds weekly ("cron: '0 1 * * TUE'", the Fedora CoreOS base) with an optional `kernel_pin` (a commented example shows pinning, e.g. `kernel_pin: 7.0.12-201.fc44.x86_64`); latest builds on merges from the `main` flavor with `image_flavors: '["main", "nvidia-open"]'`. [^16] [^17]

**Interpretation:** Upstream bluefin currently only ships the RPM Fusion latest driver via `nvidia-open`. A fork wanting 580.x must either consume `akmods-nvidia-lts` directly, or add a build variant that copies that image (as `03-install-kernel-akmods.sh` does for nvidia-open) — the tag to use is `main-45-7.3.3-300.fc45`.
**Affects:** The verdict.

**Summary of findings:** Driver 580.196.07 (2026-10-21) is the newest 580.x. Kernel 7.3.x is proven compatible (published akmods build on 7.3.3-300.fc45); no 6.x fallback. Fedora 45 is the current stable base shipping 7.3.x. Use `ghcr.io/ublue-os/akmods-nvidia-lts:main-45-7.3.3-300.fc45`, whose driver comes from negativo17's `fedora-nvidia-580` repo (currently 580.196.07-1.fc45); `NVIDIA_AKMOD_EVR` can pin an exact EVR.

## 5. Conclusion
The verdict, one line: build the fork on Fedora 45, kernel 7.3.3-300.fc45, driver 580.196.07, from `ghcr.io/ublue-os/akmods-nvidia-lts:main-45-7.3.3-300.fc45`.

### Driver
580.196.07, tagged 2026-10-21, installer present on NVIDIA's archive, and the identical version ships as `akmod-nvidia-3:580.196.07-1.fc45` in negativo17's nvidia-580 repo. No newer 580.x exists. No workflow-dispatch input pins a driver version in ublue-os/akmods; the version follows the repo, and `NVIDIA_AKMOD_EVR` pins an exact EVR when needed.

### Kernel
7.x wins outright: kernel.org's newest stable is 7.3.4 (2026-11-10), and the driver README caps nothing above "4.15 or newer". Proof of compatibility is a published build artifact, `akmods-nvidia-lts:main-45-7.3.3-300.fc45`, whose pipeline fails hard on an akmod build failure. The 6.x fallback is unnecessary and not invoked. Caveat: no 7.3.4 akmods artifact exists yet (404), so today's exact buildable pair is 580.196.07 + 7.3.3-300.fc45.

### Fedora base and ublue pieces
Fedora 45, released 2026-11-03, is the current stable and its kernel label (`-300.fc45`) matches the 7.3 series. In ublue packaging, `akmods-nvidia-lts` is the 580.x delivery vehicle: its addon script ingests `negativo17.org/repos/fedora-nvidia-580.repo` and renames it `nvidia-lts`. Upstream bluefin today only builds `nvidia-open` images (RPM Fusion 620.70.06), so a fork wanting 580.x must consume the akmods-nvidia-lts tag directly or mirror the bluefin `03-install-kernel-akmods.sh` pattern against `akmods-nvidia-lts`. Bluefin streams as of today are `stable` (weekly, CoreOS base, `coreos-stable` akmods flavor, optional `kernel_pin`) and `latest` (merge-triggered, `main` akmods flavor); no GTS stream appears in the current workflows.

## 6. Parent addendum: verification and local repo evidence (2026-11-15)

Parent independently re-verified the decisive facts (Local/Official, access date 2026-11-15):

- GHCR token-authenticated manifest probes: `ublue-os/akmods-nvidia-lts:main-45-7.3.3-300.fc45` → HTTP 200; `main-45-7.3.4-300.fc45` → HTTP 404. Confirms F3/F5 artifact existence.
- negativo17 repodata: `repomd.xml` → `d9eca9b6...-primary.xml.gz` for `nvidia-580/fedora-45/x86_64` contains `ver="580.196.07"` with `rel="1.fc45"`. Confirms F5.
- kernel.org `releases.json`: stable `7.3.4`, stable `7.2.15`, longterm `6.18.63`, mainline `7.4-rc2`. Confirms F2.

Local repo evidence from the fork clone (checkout a9236dbe, 2026-11-15), refinements for any later implementation:

1. **GTS correction to F6.** GTS is not a workflow, but the stable pipeline publishes `gts` tags as aliases. `Justfile:640-644`: `BUILD_TAGS+=("stable" "stable-${version}" ... "gts" "gts-${version}" ...)` on the Tuesday schedule or dispatch. One cadence produces both `stable` and `gts` tags, so "keep the cadence only for stable" requires deciding whether to keep or strip the gts alias tags, not removing a separate stream.
2. **Stable-stream flavor nuance.** `Justfile:123-124` maps `stable` → `akmods_flavor="coreos-stable"`, and `Justfile:150` takes the kernel from the `akmods:coreos-stable-<fedora>` label when no `kernel_pin` is set. The verified 580.x artifact is flavor `main` (`main-45-7.3.3-300.fc45`). A `coreos-stable-45-7.3.3-300.fc45` nvidia-lts tag was not probed; a stable-cadence fork must either pin the kernel to `7.3.3-300.fc45` via the `kernel_pin` workflow input (`build-image-stable.yml:34-35`) with flavor handling for nvidia-lts, or first verify a coreos-stable nvidia-lts tag exists.
3. **Hard-coded nvidia-open.** `build_files/base/03-install-kernel-akmods.sh:46` pulls `akmods-nvidia-open` only. A 580.x fork must switch this pull to `akmods-nvidia-lts` (tag scheme identical).
4. **ZFS coupling.** `build_files/base/03-install-kernel-akmods.sh:64-66` installs `akmods-zfs` at the same `${AKMODS_FLAVOR}-${fedora}-${KERNEL}` tag for coreos flavors. Pinning a kernel outside what upstream coreos-stable ships also requires a matching akmods-zfs tag.
5. **Cosign gating.** `Justfile:156-162` cosign-verifies every consumed akmods image against the ublue-os public key. Consuming existing ublue images keeps this intact; self-built akmods images would need re-signing or a key change.
6. **Cadence levers.** `.github/workflows/build-images.yml` fans out to both `build-image-stable.yml` (cron `0 1 * * TUE`, PR, merge_group) and `build-image-latest-main.yml` (PR, merge, dispatch; no cron). Stable-only cadence = drop the latest call and its workflow.

[^1]: NVIDIA, open-gpu-kernel-modules README.md at tag 580.196.07, "How to Build" section, https://raw.githubusercontent.com/NVIDIA/open-gpu-kernel-modules/580.196.07/README.md. Tier: Official. Accessed 2026-11-15.
[^2]: NVIDIA / GitHub, open-gpu-kernel-modules tags list and per-tag dates (580.196.07 = 2026-10-21T19:05:53Z), https://github.com/NVIDIA/open-gpu-kernel-modules/tags; availability probe of https://us.download.nvidia.com/XFree86/Linux-x86_64/580.196.07/NVIDIA-Linux-x86_64-580.196.07.run (HTTP 200). Tier: Official. Accessed 2026-11-15.
[^3]: kernel.org, main releases table (stable 7.3.4, 2026-11-10; longterm 6.18.63, 2026-11-10; mainline 7.4-rc2), https://www.kernel.org/. Tier: Official. Accessed 2026-11-15.
[^4]: NVIDIA, open-gpu-kernel-modules README.md at tag 580.196.07, "Supported Linux Kernel Versions", URL as [^1]. Tier: Official. Accessed 2026-11-15.
[^5]: GitHub Container Registry manifest probes: `ublue-os/akmods-nvidia-lts:main-45-7.3.3-300.fc45` (200), `ublue-os/akmods-nvidia-open:main-45-7.3.3-300.fc45` (200), `main-45-7.3.4-300.fc45` (404), via https://ghcr.io/v2/ublue-os/akmods-nvidia-lts/manifests/... Tier: Official. Accessed 2026-11-15. (Re-verified by parent, Addendum.)
[^6]: ublue-os/akmods, `.github/workflows/build-akmods-main.yml` (targets nvidia-lts/nvidia-open, version 45, dispatch input `force`), `build_files/nvidia/build-kmod-nvidia.sh` (driver selection, failure exit), `build_files/nvidia/build-ublue-os-nvidia-addons.sh` (negativo17 580 repo), main branch, https://github.com/ublue-os/akmods. Tier: Official. Accessed 2026-11-15.
[^7]: NVIDIA developer forums reports "Driver 580 does not compile/build under kernel 7.0", surfaced via web search, https://forums.developer.nvidia.com/. Tier: Third party. Accessed 2026-11-15 (snippet only; thread not opened).
[^8]: GitHub issues NVIDIA/open-gpu-kernel-modules #1507 "RTD3 silently 'Not supported' on Linux 7.3 ... 615.78.08" and #1512, https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1507. Tier: Experts/Third party. Accessed 2026-11-15.
[^9]: Fedora Magazine, "Announcing Fedora Linux 45" (stable 2026-11-03), https://fedoramagazine.org/announcing-fedora-linux-45. Tier: Official. Accessed 2026-11-15 via web search summary.
[^10]: Local machine: session environment OS Version "linux 7.3.3-300.fc45.x86_64". Tier: Local. Accessed 2026-11-15.
[^11]: ublue-os/akmods `build_files/nvidia/build-ublue-os-nvidia-addons.sh` and negativo17 repo file https://negativo17.org/repos/fedora-nvidia-580.repo. Tier: Official. Accessed 2026-11-15.
[^12]: negativo17, nvidia-580 repository metadata for Fedora 45 x86_64, repodata primary.xml (nvidia-driver and akmod-nvidia ver 580.196.07 rel 1.fc45), https://negativo17.org/repos/nvidia-580/fedora-45/x86_64/. Tier: Official. Accessed 2026-11-15. (Re-verified by parent, Addendum.)
[^13]: Local dnf repoquery against RPM Fusion nonfree-nvidia for Fedora 45: `akmod-nvidia-3:620.70.06-2.fc45.x86_64`. Tier: Local (Official repo data). Accessed 2026-11-15.
[^14]: ublue-os/bluefin, `build_files/base/03-install-kernel-akmods.sh`, main branch, https://github.com/ublue-os/bluefin. Tier: Official. Accessed 2026-11-15.
[^15]: ublue-os/bluefin `Justfile` lines ~118-126 (AKMODS_FLAVOR mapping). Tier: Official. Accessed 2026-11-15.
[^16]: ublue-os/bluefin `.github/workflows/build-image-stable.yml` (weekly cron, kernel_pin comment). Tier: Official. Accessed 2026-11-15.
[^17]: ublue-os/bluefin `.github/workflows/build-image-latest-main.yml` (image_flavors main, nvidia-open). Tier: Official. Accessed 2026-11-15.
