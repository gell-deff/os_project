# RISC-V ucore OS Project

This repository contains the group's ucore operating-system laboratory work.
Each lab is kept as a separate directory so completed labs remain reproducible
when later labs are added.

## Current lab

- `lab1`: minimal executable kernel

The current checkout is the lab1 source tree. The kernel is linked at
`0x80200000`, loaded by QEMU's built-in OpenSBI firmware, prints a startup
message, and then enters an infinite loop.

## Required tools

The lab is intended to run in a 64-bit Linux or WSL environment with:

- GNU Make
- `riscv64-unknown-elf-gcc`
- `riscv64-unknown-elf-ld`
- `riscv64-unknown-elf-objcopy`
- `riscv64-unknown-elf-objdump`
- `riscv64-unknown-elf-gdb`
- `qemu-system-riscv64`

The RISC-V toolchain's `bin` directory must be on `PATH`.

## Build and run

From the lab directory:

```bash
make
make qemu
```

QEMU runs without a graphical window. Use `Ctrl+A`, then `X` to exit.

The Makefile uses `-kernel` so that current QEMU/OpenSBI releases pass the
kernel entry address to OpenSBI. The older `-device loader,...` form shown in
some versions of the course notes may leave OpenSBI's next address as zero on
newer QEMU releases.

## Debug

Use two terminals:

```bash
make debug
```

In the second terminal:

```bash
make gdb
```

The GDB exercise follows the path from the reset address `0x1000` to the
kernel entry point `0x80200000`.

## Collaboration

The stable branch is `main`. For future labs, add a new directory such as
`lab2/` instead of overwriting an earlier lab. A completed lab should be
committed and tagged, for example:

```bash
git add .
git commit -m "完成 lab1"
git tag lab1-final
git push origin main --tags
```

Generated files under `obj/` and `bin/` are intentionally ignored. Each
collaborator still needs to install the toolchain and QEMU locally; the
repository shares source code and reports, not the operating-system
environment.
