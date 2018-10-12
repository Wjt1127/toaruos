# ToaruOS

ToaruOS is a hobbyist, educational, Unix-like operating system built entirely from scratch. It includes a kernel, bootloader, dynamic linker, C standard library, composited windowing system, and several utilities and applications. All components of the core operating system are original, providing a complete environment in approximately 60,000 lines of C and assembly, all of which is included in this repository.

![Screenshot](https://i.imgur.com/aCvPH4X.png)
*Demonstration of ToaruOS's UI, terminal emulator, and editor, showing a Python graphical application with Freetype and Cairo.*

## History

The ToaruOS project began in December 2010 and has its roots in an independent student project. The goals of the project have changed throughout its history, initially as a learning experience for the authors, and more recently as a complete, from-scratch ecosystem.

ToaruOS is currently in the 1.6.x series, which represents the merging of a project (the "NIH" branch) to replace Newlib, the third-party C standard library employed by earlier versions of the operating system, with a new, in-house libc.

## Features

Early in its development, ToaruOS was focused on providing a modern *graphical user interface* built on top of an efficient *composited window manager*. More recently, that focus has shifted towards providing a more compliant *Unix terminal environment*, including an Xterm-alike *graphical terminal emulator* (with support for several escape sequences including 24-bit color and alternate screen buffers), several utilities, and a shell (though it is not very POSIX-y).

### Notable Components

- **Toaru Kernel**, [kernel/](kernel/), the core of the operating system. 
- **Yutani**  (window compositor), [apps/compositor.c](apps/compositor.c), manages window buffers, layout, and input routing.
- **Bim** (text editor), [apps/bim.c](apps/bim.c), is a vim-inspired editor with syntax highlighting.
- **Terminal**, [apps/terminal.c](apps/terminal.c), xterm-esque terminal emulator with 256 and 24-bit color support.
- **ld.so** (dynamic linker/loader), [linker/linker.c](linker/linker.c), loads dynamically-linked ELF binaries.
- **Esh** (shell), [apps/sh.c](apps/sh.c), supports pipes, redirections, variables, and more.

## Current Goals

The following projects are currently in progress:

- **Complete the re-porting of Binutils** to allow for some basic assembly development and to support the eventual return of GCC.
- **Improve POSIX coverage** especially in regards to pipelines, process groups, synchronization primitives, as well as by providing more common utilities.
- **Continue to improve the C library** which remains quite incomplete compared to Newlib and is a major source of issues with bringing back old ports.
- **Implement a native dynamic, interpreted programming language** to replace Python, which was used prior to ToaruOS 1.6.x to provide most of the desktop environment.

## Building / Installation

To build ToaruOS from source, it is currently recommended you use a recent Debian- or Ubuntu-derived Linux host environment.

Several packages are necessary: `build-essential`, `yasm`, `xorriso`, `genext2fs`, `python3`, `mtools`, `gnu-efi`

Beyond package installation, no part of the build needs root privileges. 

The build process has two parts: building a cross-compiler, and building the operating system. The cross-compiler uses GCC 6.4.0 and will be built automatically by `make` if other dependencies have been met. This only needs to be done once, and the cross-compiler does not depend on any of the components built for the operating system itself, though it is attached to the base directory of the repository so you may need to rebuild the toolchain if you move your checkout. Once the cross-compiler has been built, `make` will continue to build the operating system itself.

### Build Process Internals

The `Makefile` first checks to see if a toolchain is available and activates it (appends its `bin` directory to `$PATH`). If a toolchain is not available, users are prompted if they would like to build one. This process downloads and patches both Binutils and GCC and then builds and installs them locally.

The `Makefile` then uses a Python tool, `auto-dep.py`, to generate additional Makefiles for the userspace applications libraries, automatically resolving dependencies based on `#include` directives.

In an indeterminate order, C library, kernel, modules, userspace librares and applications are built. Three boot loaders (one BIOS ATAPI CD loader for emulators, and both a 32-bit and 64-bit EFI loader for general use) are then built. Deployed binaries are stored in `base` which is converted into an EXT2 filesystem image with `genext2fs`. This image, along with the bootloader files and kernel are then placed in `fatbase` which is converted into a FAT image for use as the EFI boot payload. That image is then placed in `cdbase` along with shadow files representing each of the files in the FAT image, and `cdbase` is compiled into an ISO 9660 CD El Torito image. The CD image is then passed through a tool to map the shadow files to their actual data from the FAT image, creating a hybrid ISO 9660 / FAT.

### Clang

The kernel and driver modules have been successfully built with Clang using the `i686-elf` target. If you would like to experiment with using Clang to build ToaruOS, pass `USE_CLANG=1` to `make` from a clean build. You may confirm that Clang was used to build the kernel by examining `/proc/compiler` within the OS.

### Third-Party Components

Prior to ToaruOS 1.6.x, many third-party components were included by default (Python, libpng, zlib, Cairo, freetype, and so on). These are no longer part of the default distribution or build process and must be built manually. Complete guides for building these components are currently being drafted. The instructions for building Python are complete and [available from the wiki](https://github.com/klange/toaruos/wiki/How-to-Python) (note that a host installation of Python 3.6 is required to build Python 3.6 and satisfying this is left as an exercise to the reader).

Freetype and Cairo have also been successfully built under the new in-house C library. When either of these are available, optional extension bindings may be built with `make ext-freetype` and `make ext-cairo` respectively. When the Freetype extension binding is available in the OS, alongside required Truetype font files, Freetype will be used to render text in several applications (including the terminal emulator, menus, and window decorations). When the Cairo extension binding is available, it is used by the compositor to provide improved performance.

### Project Layout

- **apps** - Userspace applications, all first-party.
- **base** - Ramdisk root filesystem staging directory. Includes C headers in `base/usr/include`, as well as graphical resources for the compositor and window decorator.
- **boot** - Bootloader, including BIOS and EFI IA32 and X64 support.
- **cdrom** - Staging area for ISO9660 CD image, containing mostly blank shadow files for the FAT image.
- **ext** - Optional runtime-loaded bindings for third-party libraries.
- **fatbase** - Staging area for FAT image used by EFI.
- **kernel** - The Toaru kernel.
- **lib** - Userspace libraries.
- **libc** - C standard library implementation.
- **linker** - Userspace dynamic linker/loader, implements shared library support.
- **modules** - Kernel modules/drivers.
- **util** - Utility scripts, staging directory for the toolchain (binutils/gcc).
- **.make** - Generated Makefiles.


## Running ToaruOS

It is highly recommended that interested users run ToaruOS from virtual machines. While we have done some testing on real hardware, driver support is still limited and virtual machines provide easily tested environments where we can guarantee some level of useful functionality.

QEMU and VirtualBox are recommended and provide the most functonality. Audio support is not yet available in VMware. In VirtualBox and VMware, automatic guest display resizing is available (and a tool is available to provide similar functionality in QEMU). All three of the major VMs also support absolute mouse input.

### QEMU

1GB of RAM and an Intel AC'97 sound chip are recommended:

```
qemu-system-i386 -cdrom image.iso -serial mon:stdio -m 1G -soundhw ac97,pcspk -enable-kvm -rtc base=localtime
```

You may also use OVMF with the appropriate QEMU system target. Our EFI loader supports both IA32 and X64 EFIs:

```
qemu-system-x86_64 -cdrom image.iso -serial mon:stdio -m 1G -soundhw ac97,pcspk -enable-kvm -rtc base=localtime \
  -bios /usr/share/qemu/OVMF.fd
```

```
qemu-system-i386 -cdrom image.iso -serial mon:stdio -m 1G -soundhw ac97,pcspk -enable-kvm -rtc base=localtime \
  -bios /path/to/OVMFia32.fd
```

Additionally, a tool is available for running QEMU, under specific environments, with automatic support for resizing the guest display resolution when the QEMU window changes size: `util/qemu-harness.py`

### VirtualBox

ToaruOS should function either as an "Other/Unknown" guest or an "Other/Unknown 64-bit" guest with EFI.

All network chipset options should work except for `virtio-net` (work on virtio drivers has not yet begun).

It is highly recommended, due to the existence of Guest Additions drivers, that you provide your VM with at least 32MB of video memory to support larger display resolutions - especially if you are using a 4K display.

Ensure that the audio controller is set to ICH AC97 and that audio output is enabled (as it is disabled by default in some versions of VirtualBox).

Keep the system chipset set to PIIX3 for best compatibility. 1GB of RAM is recommended.

### VMWare

Support for VMWare is experimental, though it has improved significantly in recent months. Optional support is provided for VMware's automatic guest display sizing, which is enabled by default and can be disabled from the bootloader menu.

- Create a virtual machine for a 64-bit guest. (ToaruOS is 32-bit, but this configuration selects some hardware defaults that are desirable)
- Ensure the VM has 1GB of RAM.
- It is recommended you remove the hard disk and the audio device.
- For network settings, the NAT option is recommended.

### Bochs

Using Bochs to run ToaruOS is not advised; however the following configuration options are recommended if you wish to try it:

- Attach the CD and set it as a boot device.
- Ensure that the `pcivga` device is enabled or ToaruOS will not be able to find the video card through PCI.
- Provide at least 512MB of RAM to the guest.
- If available, enable the `e1000` network device using the `slirp` backend.
- Clock settings of `sync=realtime, time0=local, rtc_sync=1` are recommended.

## Community

### Mirrors

ToaruOS is regularly mirrored to multiple Git hosting sites.

- Gitlab: [toaruos/toaruos](https://gitlab.com/toaruos/toaruos)
- GitHub: [klange/toaruos](https://github.com/klange/toaruos)
- Bitbucket: [klange/toaruos](https://bitbucket.org/klange/toaruos)
- ToaruOS.org: [klange/toaruos](https://git.toaruos.org/klange/toaruos)

### IRC

`#toaruos` on Freenode (`irc.freenode.net`)

## FAQs

### Is ToaruOS self-hosting?

Prior to the merging of the "NIH" branch, ToaruOS was capable of running Binutils and GCC and could build its own kernel and core userspace (and also had a port of Bochs in which to run the resulting images), but was not demonstrated to have been able to build GCC and Binutils themselves due to limitations in the native shell. Native GCC builds for ToaruOS 1.6.x are experimental, but functioning. Consideration has also been put towards development of our own C compiler.

### Is ToaruOS a Linux distribution?

As stated several times in this document, ToaruOS's kernel is written entirely from scratch. In addition to legal restrictions related to Linux's and ToaruOS's licensing terms, the very goal of the project prevents us from taking code from Linux in any way. Beyond that, the entire core userspace, all drivers, and the bootloader are also built from scratch. ToaruOS is its own project in all ways beyond the look and feel of its APIs and tools, which are based on Unix.

### Are there plans for a 64-bit port / SMP support?

With the development of ToaruOS's "NIH" branch, a secondary goal in removing third-party dependencies was to make the operating system more viable for a 64-bit port. That said, the actual development of a 64-bit kernel is currently on pause while other goals are pursued. Due to the limited size of the development team, it is not feasible to continue work on the 64-bit kernel at this time.

SMP support likely poses a larger challenge as the early toy design for ToaruOS did not take into account multiprocessor systems and thus many challenges exist in getting the kernel to a functioning state with SMP.
