# Fork: DarkGhostHunter/bluefin

This is a fork of [ublue-os/bluefin](https://github.com/ublue-os/bluefin) that ships only the **stable** stream, rebuilt every Tuesday at 01:00 UTC (plus `stable-daily` on merges to `main`), on Fedora 45 with the kernel pinned to **7.3.3-300.fc45.x86_64** and, for NVIDIA systems, the proprietary **NVIDIA 580.x LTS driver** (currently 580.196.07, delivered through [ublue-os/akmods-nvidia-lts](https://github.com/ublue-os/akmods) images fed by negativo17's `fedora-nvidia-580` repository).

## Images

| Image | Use it if |
| --- | --- |
| `ghcr.io/darkghosthunter/bluefin:stable` | Standard Bluefin, no NVIDIA driver |
| `ghcr.io/darkghosthunter/bluefin-dx:stable` | Standard Bluefin with developer tools |
| `ghcr.io/darkghosthunter/bluefin-nvidia-lts:stable` | NVIDIA GPU (Maxwell/GTX 900 or newer) |
| `ghcr.io/darkghosthunter/bluefin-dx-nvidia-lts:stable` | NVIDIA GPU + developer tools |

Every image also carries a dated tag like `stable-45.20261115` if you want to pin to a specific weekly build (first published: `45.20261115`, `stable-45.20261115`, `stable-20261115`, 2026-11-15). Pick `bluefin-nvidia-lts` if your machine has an NVIDIA GPU; otherwise pick `bluefin`.

## Rebase from an existing Bluefin or Fedora Silverblue

 rpm-ostree systems (Bluefin, Silverblue):

```bash
sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/darkghosthunter/bluefin-nvidia-lts:stable
```

For the non-NVIDIA image, use `ghcr.io/darkghosthunter/bluefin:stable` instead. After the first reboot you can rebase again through the signed transport:

```bash
sudo rpm-ostree rebase ostree-remote-image:ostree-remote-signed:registry:ghcr.io/darkghosthunter/bluefin-nvidia-lts:stable
```

The signed transport requires this fork's `cosign.pub` configured in `/etc/ostree/remotes.d`; until then, the `ostree-unverified-registry:` command above is the supported entry point (transport-level HTTPS only; the image payload is identical).

 bootc systems:

```bash
sudo bootc switch ghcr.io/darkghosthunter/bluefin-nvidia-lts:stable
```

## Caveats

- The NVIDIA flavor uses the **proprietary kernel module**, so it supports **Maxwell (GTX 900) and newer** GPUs. No open-source `nvidia-open` flavor is published on the stable stream.
- The driver tracks the **580.x LTS series** from negativo17 via ublue `akmods-nvidia-lts` (currently **580.196.07**) and moves within 580.x on weekly rebuilds. Exact driver versions are not pinned.
- **ZFS is NOT included.** Upstream stable ships it from the `coreos-stable` akmods line; this fork builds on the `main` akmods line to get the 7.3 kernel, and ZFS artifacts do not exist there.
- The kernel stays pinned at `7.3.3-300.fc45.x86_64` until the maintainers bump it (see the comment in `.github/workflows/build-image-stable.yml`).
- Images are signed with this repository's cosign key (`cosign.pub` at the repo root).

## Verify an image

```bash
cosign verify --key cosign.pub ghcr.io/darkghosthunter/bluefin-nvidia-lts:stable
```

The kernel pin and akmods/NVIDIA tag pairing are backed by the investigation in [docs/nvidia-580-kernel-investigation.md](docs/nvidia-580-kernel-investigation.md).
