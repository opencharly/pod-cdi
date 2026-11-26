# AGENTS.md — pod-cdi

The org rulebook at the umbrella (`opencharly/opencharly` `AGENTS.md`) applies in full.
This file adds only what is specific to this repo.

## This repo holds the POD contract

- **`pod-cdi` (here)** — the boxes it contains are pods. The driver comes from the host via CDI.
  Composes `layer-cdi` and nothing nvidia-baked.
- **A GPU VM with passthrough** — the guest owns its kernel module, so the driver user-space
  is baked into the image. Such an image composes `layer-nvidia`.

Never compose `layer-nvidia` here. `layer-cdi`'s plan steps enforce that: they fail the
build when a host-coupled driver library or nvidia binary is baked in. If a change needs
`nvidia-utils`, `LD_LIBRARY_PATH`, or `nvidia-ctk` in the image, the change belongs in a VM
image, not here.

## Adding a consumer

A consumer is a box in another repo that sets `base:` to this box through an `import:`
of `@github.com/opencharly/pod-cdi:<tag>`. Pin a **tag** — a merged ref. A branch or SHA
pin does not resolve for `@github` candy refs (opencharly/charly#1201).
