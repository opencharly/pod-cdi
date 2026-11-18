# layer-cdi

The CDI runtime contract for GPU images.

A GPU container does not get its driver from the image. The kernel module is host-global, so the
driver user-space must be injected per container (`--device nvidia.com/gpu=all`) by the NVIDIA
container toolkit. An image that bakes its own driver user-space wins the SONAME resolution against
the injected files and produces `Driver/library version mismatch` — and consumers that size their
work from NVML (ollama, for one) silently fall back to CPU.

`candy/cdi` therefore removes what a package manager installs as a side effect, proves at build time
that nothing remains, and proves at check time that the container resolved the injected driver at the
running kernel module's version. Every GPU image composes it.

See `.agents/skills/cdi/SKILL.md`.
