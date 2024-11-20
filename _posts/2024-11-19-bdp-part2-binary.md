---
layout: post
title: "Blu-ray disc player: part 2 - compiling ARMv6 binary"
date: 2024-11-19 23:50:00 +0000
categories: pet_project
tags: linux, arm
excerpt: How to create C binary by hacking compiler front-end and linker
---

Last time, we take a look at the Blu-ray disc player and discovered that we can create a shared library and put it in place of another debugging library to do a code execution. Our objective is to compile a binary that can run on this player. We also saw that the binaries in its firmware are compiled against ARMv6KZ, and can use hard-float. After some extra [research](https://reviews.llvm.org/D18086), I realized they in fact are compiled by GCC at least with flag `-march=armv6z` (with the K implicitly-defined). The evidence is in the ELF metadata, where *Tag_CPU_arch* is `v6KZ` as we know, but *Tag_CPU_name* is simply `"6Z"`. For consistency, all the following commands assume using PowerShell as shell. If you would like to use for bash or other Unix-like shell, change `` ` `` (backtick) to `\` (backslash) for new line.

Here is the code I will try to compile (based on the [blog post](http://www.malcolmstagg.com/bdp/firmware-less.html)):

```c
#include <stdlib.h>

__attribute__((constructor))
void libSample() {
    unsetenv("LD_PRELOAD");
    system("/mnt/sda1/script.sh");
}
```

The code will execute _script.sh_ in the root of USB drive. As quick as I could, I grabbed the latest `arm-none-linux-gnueabihf` Arm GNU toolchain for Windows on [developer.arm.com](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain), extracted and ran:

```powershell
arm-none-linux-gnueabihf-gcc -c -fPIC test.c -march=armv6z -o libSegFault.o
```

```sh
In file included from D:/tools/arm-gnu-toolchain-13.3.rel1-mingw-w64-i686-arm-none-linux-gnueabihf/arm-none-linux-gnueabihf/libc/usr/include/endian.h:35,
                 from D:/tools/arm-gnu-toolchain-13.3.rel1-mingw-w64-i686-arm-none-linux-gnueabihf/arm-none-linux-gnueabihf/libc/usr/include/sys/types.h:176,
                 from D:/tools/arm-gnu-toolchain-13.3.rel1-mingw-w64-i686-arm-none-linux-gnueabihf/arm-none-linux-gnueabihf/libc/usr/include/stdlib.h:514,
                 from test.c:1:
D:/tools/arm-gnu-toolchain-13.3.rel1-mingw-w64-i686-arm-none-linux-gnueabihf/arm-none-linux-gnueabihf/libc/usr/include/bits/byteswap.h: In function '__bswap_16':
D:/tools/arm-gnu-toolchain-13.3.rel1-mingw-w64-i686-arm-none-linux-gnueabihf/arm-none-linux-gnueabihf/libc/usr/include/bits/byteswap.h:35:1: sorry, unimplemented: Thumb-1 'hard-float' VFP ABI
   35 | {
      | ^
```

For the first time in my life, I received a nice but funny compiler error: _sorry, unimplemented_. You're a compiler, how could this happen? But StackOverflow [post](https://stackoverflow.com/questions/35132319/build-for-armv6-with-gnueabihf) saves the day again. For a long time, as far back as Linaro times, ARM toolchain has been compiled with `--with-arch=armv7-a`, and this means only ARMv7-A, Thumb-2 is available. Great.

What if we use Clang to generate code instead? Shouldn't be too hard:

```powershell
clang --target=arm-none-linux-gnueabihf `
  --sysroot=D:\tools\arm-gnu-toolchain-13.3.rel1-mingw-w64-i686-arm-none-linux-gnueabihf\arm-none-linux-gnueabihf\libc `
  -march=armv6z+fp -mfloat-abi=hard -mfpu=vfpv2 `
  test.c `
  -c -fPIC -o libSegFault.o
```

No error this time. Let's check out `readelf`:

```yaml
Attribute Section: aeabi
File Attributes
  Tag_conformance: "2.09"
  Tag_CPU_name: "arm1176jzf-s"
  Tag_CPU_arch: v6
  Tag_ARM_ISA_use: Yes
  Tag_THUMB_ISA_use: Thumb-1
  Tag_FP_arch: VFPv2
  Tag_ABI_PCS_R9_use: V6
  Tag_ABI_PCS_RW_data: PC-relative
  Tag_ABI_PCS_RO_data: PC-relative
  Tag_ABI_PCS_GOT_use: GOT-indirect
  Tag_ABI_PCS_wchar_t: 4
  Tag_ABI_FP_denormal: Needed
  Tag_ABI_FP_exceptions: Unused
  Tag_ABI_FP_number_model: IEEE 754
  Tag_ABI_align_needed: 8-byte
  Tag_ABI_align_preserved: 8-byte, except leaf SP
  Tag_ABI_enum_size: int
  Tag_ABI_VFP_args: VFP registers
  Tag_ABI_optimization_goals: Aggressive Debug
  Tag_CPU_unaligned_access: None
  Tag_ABI_FP_16bit_format: IEEE 754
  Tag_Virtualization_use: TrustZone
```

Close enough! Also notice that here is just code generation, so we only used header files (`include/`) from the sysroot. You can technically swap any header files with minimal amount of issues in the outcome.

But now we still need to link the binary using linker into _.so_ file. This is actually not easy at all. I tried to call both `clang` and `gcc` with specifying the location of the source code, but both failed to find the correct path for _crtXXX.o_ files. What are these files, you ask? _crt_ stands for C Run-Time, and these are necessary to initialize the environment before running a C program. If you have done reverse-engineering on a Linux binary and saw [`libc_start_main`](https://stackoverflow.com/questions/62709030/what-is-libc-start-main-and-start), this is where the function is from. `_start` is the actual entry point of the executable and calls `libc_start_main`, which would in turn call `main`.

Linking order is also important, because it is the order of the code inside the binary when the sections are merged. This StackOverflow [question](https://stackoverflow.com/questions/22160888/what-is-the-difference-between-crtbegin-o-crtbegint-o-and-crtbegins-o) describes exactly what we are wanting to do, and it has the answer. Just to be sure, I also run both `clang` & `gcc` with `-v` to see the parameter used for `ld` when linking. The general linking command is (assuming using a cross-compiler toolchain that has file structure similar to one built using crosstool-ng):

```powershell
ld --sysroot=... `
  -EL -X --hash-style=gnu --eh-frame-hdr -m armelf_linux_eabi `
  -shared -o libSegFault.so `
  crti.o crtbeginS.o `
  -Llib/gcc/$target/$gcc_ver `
  -Llib/gcc -L$target/lib `
  -L$target/sysroot/lib `
  -L$target/sysroot/usr/lib `
  libSegFault.o `
  -lgcc `
  --as-needed -lgcc_s `
  --no-as-needed -lc -lgcc `
  --as-needed -lgcc_s `
  --no-as-needed `
  crtendS.o crtn.o
```

`$target` is the target triple of the toolchain, `$gcc_ver` is the version of gcc of the toolchain. Note that we need CRT compatible with ARMv6, and the previous GCC toolchain is compiled against ARMv7-A, so here I use [`armv6-rpi-linux-gnueabihf` GCC toolchain](https://github.com/tttapa/docker-arm-cross-toolchain) as the sysroot instead. Although that toolchain is compiled for Linux, we can reuse `ld.exe` found in the Arm GNU toolchain from earlier since linking is not architecture-dependant. Below is the command line I ran:

```powershell
ld `
  --sysroot=D:/tools/armv6-rpi-linux-gnueabihf/armv6-rpi-linux-gnueabihf/sysroot `
  -EL -X --hash-style=gnu --eh-frame-hdr -m armelf_linux_eabi `
  -shared -o libSegFault.so `
  "D:/tools/armv6-rpi-linux-gnueabihf/armv6-rpi-linux-gnueabihf/sysroot/usr/lib/crti.o" `
  "D:/tools/armv6-rpi-linux-gnueabihf/lib/gcc/armv6-rpi-linux-gnueabihf/14.2.0/crtbeginS.o" `
  -LD:/tools/armv6-rpi-linux-gnueabihf/lib/gcc/armv6-rpi-linux-gnueabihf/14.2.0 `
  -LD:/tools/armv6-rpi-linux-gnueabihf/lib/gcc `
  -LD:/tools/armv6-rpi-linux-gnueabihf/armv6-rpi-linux-gnueabihf/lib `
  -LD:/tools/armv6-rpi-linux-gnueabihf/armv6-rpi-linux-gnueabihf/sysroot/lib `
  -LD:/tools/armv6-rpi-linux-gnueabihf/armv6-rpi-linux-gnueabihf/sysroot/usr/lib `
  libSegFault.o `
  -lgcc `
  --as-needed -lgcc_s `
  --no-as-needed -lc -lgcc `
  --as-needed -lgcc_s `
  --no-as-needed `
  "D:/tools/armv6-rpi-linux-gnueabihf/lib/gcc/armv6-rpi-linux-gnueabihf/14.2.0/crtendS.o" `
  "D:/tools/armv6-rpi-linux-gnueabihf/armv6-rpi-linux-gnueabihf/sysroot/usr/lib/crtn.o"
```

Now we got the binary with linker compiler error. Double check using `readelf`:

```yaml
Attribute Section: aeabi
File Attributes
  Tag_CPU_name: "6"
  Tag_CPU_arch: v6
  Tag_ARM_ISA_use: Yes
  Tag_THUMB_ISA_use: Thumb-1
  Tag_FP_arch: VFPv2
  Tag_ABI_PCS_GOT_use: GOT-indirect
  Tag_ABI_PCS_wchar_t: 4
  Tag_ABI_FP_denormal: Needed
  Tag_ABI_FP_exceptions: Needed
  Tag_ABI_FP_number_model: IEEE 754
  Tag_ABI_align_needed: 8-byte
  Tag_ABI_align_preserved: 8-byte, except leaf SP
  Tag_ABI_enum_size: int
  Tag_ABI_VFP_args: VFP registers
  Tag_CPU_unaligned_access: v6
  Tag_ABI_FP_16bit_format: IEEE 754
  Tag_Virtualization_use: TrustZone
```

Sounds good! Let's setup file and folder inside the USB drive for code execution:

```
.
└───bbb
    └───libSegFault.so
└───script.sh
```

And the content of _script.sh_ is:

```sh
#!/bin/sh
echo root:admin | chpasswd
killall telnetd
telnetd -p 42001 &
```

Now, we turn on the player, open browser. After seeing the USB drive flashing (there's a read!), we can `telnet` in, and behold:

```
~ # cat /proc/cpuinfo
Processor       : ARMv6-compatible processor rev 7 (v6l)
BogoMIPS        : 804.86
Features        : swp half thumb fastmult vfp edsp java
CPU implementer : 0x41
CPU architecture: 7
CPU variant     : 0x0
CPU part        : 0xb76
CPU revision    : 7

Hardware        : mt85xx
Revision        : 0000
Serial          : 0000000000000000
```

```
~ # cat /proc/version
Linux version 2.6.35 (merger@rvds105) (gcc version 4.5.1 (GCC) ) #1 PREEMPT Fri Jan 20 12:08:15 JST 2012
```

This is just the beginning of the fun. Next time, we will take a look at how to cross-compile any C program for this platform.
