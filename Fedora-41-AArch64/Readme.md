# Fedora 41 Images

This set of images is based on the Fedora 41 image.
It has three flavors, `build`, `test`, and `dev`.
The first two are primarily intended for automated builds
and CI usage.

The `build` image contains the compilers and build tools
needed for building EDK2 under Linux (AArch64).

The `test` image extends the `build` image and adds Qemu for
testing purposes.

The `dev` image in turn extends the `test` image and adds developer
convenience tools, for example the git credential manager.

These images include:
- gcc 14.2.1 (x86, arm, aarch64, riscv, loongarch64)
- nasm 2.16.03
- Python 3.13.3
- Qemu 9.1.3 (x86, arm, aarch64, riscv, loongarch64)
