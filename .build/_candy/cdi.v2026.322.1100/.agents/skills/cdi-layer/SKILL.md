---
name: cdi-layer
description: |-
  The `cdi` CANDY itself — what it removes, what it deliberately keeps (the ICD JSONs), the
  build-time assertion that cannot be fooled, and the runtime check that catches a shadowed
  injection.
family: distros
---

# The `cdi` candy

One plan, three parts, in this order:

| Step | Kind | Why it lives there |
|---|---|---|
| `cdi-drop-baked-driver` | `run:` (root) | Removes the driver user-space and the `nvidia*` binaries a package install pulled in. |
| `cdi-no-baked-driver-libs` / `cdi-no-baked-driver-binaries` | `check:` (`deploy`) | Asserts the removal held — files AND links AND versioned files, and the whole `nvidia*` binary set. |
| `cdi-injected-driver-matches-module` | `check:` (`runtime`) | Asserts the container resolved the injected driver and that its version equals `/proc/driver/nvidia/version`. |

## Editing rules

- **Never narrow the patterns to a hand-listed subset.** This candy's first revision did, and its own
  assertion immediately caught three leftovers it had missed (`nvidia-container-toolkit` and its two
  `nvidia-container-runtime.<variant>` wrappers) — the package also ships `nvidia-ngx-updater`,
  `nvidia-pcc`, `nvidia-powerd`, `nvidia-sleep.sh`.
- **Never move the build-time assertion into a `check:`.** A GPU-attached container makes the
  injected libraries indistinguishable from baked ones at the same paths.
- **Keep the ICD JSONs.** They name SONAMEs, so the injected links satisfy them; deleting them breaks
  Vulkan/OpenCL for no gain.
- **The runtime probe must stay version-based, not existence-based.** "the file exists" and "CUDA
  device detected" both pass on a shadowed image.

## Debugging a `cdi-injected-driver-matches-module` failure

1. `readlink -f /usr/lib/libnvidia-ml.so.1` — if it names a version other than
   `/proc/driver/nvidia/version`, an image library survived.
2. `find /usr/lib /usr/lib64 -maxdepth 1 -name 'libnvidia-*.so*'` — a leftover here means a LATER
   layer reinstalled the package; find which and have it compose `cdi` after its installs.
3. On the host: `charly doctor` reports the CDI mechanism and the host's own userspace/module pair.
