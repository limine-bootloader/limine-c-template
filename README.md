# Limine C Template

This repository will demonstrate how to set up a basic kernel in C using Limine.

## How to use this?

### Dependencies

Any `make` command depends on GNU make (`gmake`) and is expected to be run using it. This usually means using `make` on most GNU/Linux distros, or `gmake` on other non-GNU systems.

It is recommended to build this project using a standard UNIX-like system, using a Clang/LLVM toolchain capable of cross compilation.

Building also requires `git`, used to fetch the kernel's dependencies (see `kernel/get-deps`), `curl`, used to download the Limine release and the EDK2 OVMF firmware images, and a C compiler for the host (`cc` by default, see the `HOST_CC` `make` variable), used to build the `limine` host utility.

Additionally, building an ISO with `make all` requires `xorriso`, and building a HDD/USB image with `make all-hdd` requires `sgdisk` (usually from `gdisk` or `gptfdisk` packages) and `mtools`.

Assembly files with the `*.S` extension are built using the same toolchain as the C sources. Only `*.asm` files, which are built for `x86_64` alone and of which the template ships none, require `nasm`. The `run` targets require `qemu`.

### Toolchain selection

The `TOOLCHAIN` and `TOOLCHAIN_PREFIX` `make` variables can be used to set the toolchain. `TOOLCHAIN` can be set to `llvm` to use Clang/LLVM.

For example:
```
make TOOLCHAIN=llvm
```
or:
```
make TOOLCHAIN_PREFIX=x86_64-elf-
```

### Architectural targets

The `ARCH` make variable determines the target architecture to build the kernel and image for.

The default `ARCH` is `x86_64`. Other options include: `aarch64`, `loongarch64`, and `riscv64`.

### Makefile targets

Running `make all` will compile the kernel (from the `kernel/` directory) and then generate a bootable ISO image.

Running `make all-hdd` will compile the kernel and then generate a raw image suitable to be flashed onto a USB stick or hard drive/SSD.

Running `make run` will build the kernel and a bootable ISO (equivalent to make all) and then run it using `qemu` (if installed).

Running `make run-hdd` will build the kernel and a raw HDD image (equivalent to make all-hdd) and then run it using `qemu` (if installed).

For x86_64, the `run-bios` and `run-hdd-bios` targets are equivalent to their non `-bios` counterparts except that they boot `qemu` using the default SeaBIOS firmware instead of OVMF.
