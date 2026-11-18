# pod-cdi — the CDI GPU **pod** base box

The container contract for GPU workloads, and the pod sibling of
[opencharly/vm-nvidia](https://github.com/opencharly/vm-nvidia).

## The boundary this repo exists to hold

A GPU **container** does not own its driver. The kernel module is host-global, so the
driver user-space must come from the **host**, injected per container by the container
runtime through the NVIDIA container toolkit's CDI spec
(`/etc/cdi/nvidia.yaml`, generated on the host by `nvidia-ctk cdi generate`) and its
runtime hook. `nvidia-container-toolkit` and `nvidia-ctk` are **host tools**; they never
execute inside a container.

A GPU **VM** with passthrough is the opposite: the guest owns its kernel module, so the
driver user-space must be **baked into the image**.

Those are two different images, and mixing them produces the mess this split removes —
an image that bakes a driver user-space which shadows the host's, and whose build
asserts a host tool is present inside a container.

| | pod-cdi (this repo) | vm-nvidia |
|---|---|---|
| substrate | pod / container | VM with GPU passthrough |
| driver source | host, injected via CDI | baked into the image |
| layer | `layer-cdi` | `layer-nvidia` |
| `nvidia-ctk` in image | **no** | yes |
| `nvidia-utils` / `LD_LIBRARY_PATH` | **no** | yes |

## Composition

```yaml
pod-cdi:
  candy:
    base: cachyos.cachyos
    candy:
      - '@github.com/opencharly/layer-agent-forwarding:v2026.274.0608'
      - '@github.com/opencharly/layer-cdi:v2026.322.1100'
```

`layer-cdi` owns the contract and enforces it: two of its three plan steps fail the
build if a host-coupled driver library or nvidia binary is baked in.

## Measured

A pod on this contract serves CUDA with **no** `/usr/bin/nvidia-ctk` in its image:

```
[INFO] Total VRAM 16027 MB, total RAM 126362 MB
[INFO] Device: cuda:0 NVIDIA GeForce RTX 4080 SUPER : cudaMallocAsync
```

while the host carries the toolkit and the spec:

```
$ command -v nvidia-ctk
/usr/bin/nvidia-ctk
$ ls -l /etc/cdi/nvidia.yaml
-rw-r--r-- 1 root root 27076 … /etc/cdi/nvidia.yaml
```

## Consumers

`box/comfyui`, `box/versa`, `box/selkies-kde-nvidia`, `box/selkies-labwc-nvidia`
in opencharly/distro-cachyos — all pods, all on this contract.
