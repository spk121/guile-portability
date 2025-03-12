# guile-portability
A meta-project to hold my Guile build scripts and notes.

This project contains scripts and documents for building GNU Guile in
various ways, OSs, and container formats.

A significant goal is to have GNU Guile work well on Microsoft
Windows,status for the benefit of LilyPond and for the Lisp Game Jam.

## Project Docs

1. [Overview and Status](docs/overview.md)
2. [HOWTO: Email on Emacs](docs/email.md)
3. [HOWTO: Debbugs](docs/debbugs.md)
4. [HOWTO: IRC](docs/irc.md)

## Badges

| Project | Status | Notes |
|---------|--------|-------|
| [Ubuntu](https://github.com/spk121/guile-portability/actions/workflows/ubuntu.yml) | ![Ubuntu](https://github.com/spk121/guile-portability/actions/workflows/ubuntu.yml/badge.svg) | default ./configure |
| [Ubuntu Distcheck](https://github.com/spk121/guile-portability/actions/workflows/ubuntu-distcheck.yml) |  ![Build Status](https://raw.githubusercontent.com/spk121/badges/spk121/guile-portability/master/build.svg?sanitize=true) | |
| [Ubuntu Basic](https://github.com/spk121/guile-portability/actions/workflows/ubuntu.yml) | ![Ubuntu](https://github.com/spk121/guile-portability/actions/workflows/ubuntu.yml/badge.svg) | Mini-GMP, No Threads, No JIT, No LTO |
| [Debian 64-bit in Docker]() | | |
| [Debian 32-bit in Docker]() | | |
| [FlatPak]() | | |
| [Cygwin](https://github.com/spk121/guile-portability/actions/workflows/cygwin.yml) | ![Cygwin](https://github.com/spk121/guile-portability/actions/workflows/cygwin.yml/badge.svg) | |
| [Cygwin Distcheck](https://github.com/spk121/guile-portability/actions/workflows/cygwin-distcheck.yml) | ![Cygwin Distcheck](https://github.com/spk121/guile-portability/actions/workflows/cygwin-distcheck.yml/badge.svg) | |
| [MSYS](https://github.com/spk121/guile-portability/actions/workflows/msys.yml) | ![MSYS](https://github.com/spk121/guile-portability/actions/workflows/msys.yml/badge.svg) | |
| [MinGW-w32 Basic](https://github.com/spk121/guile-portability/actions/workflows/mingw-w32-basic.yml) | ![MinGW](https://github.com/spk121/guile-portability/actions/workflows/mingw-w32-basic.yml/badge.svg) | MSVCRT, Mini-GMP, No Threads, No JIT, No LTO, Gnu Filesystem Hierarchy |
| [MinGW-w64 Basic](https://github.com/spk121/guile-portability/actions/workflows/mingw-w64-basic.yml) | ![MinGW](https://github.com/spk121/guile-portability/actions/workflows/mingw-w64-basic.yml/badge.svg) | UCRT, Mini-GMP, No Threads, No JIT, No LTO, Gnu Filesystem Hierarchy | 
| [MinGW-w64 Full](https://github.com/spk121/guile-portability/actions/workflows/mingw-w64.yml) | ![MinGW](https://github.com/spk121/guile-portability/actions/workflows/mingw-w64.yml/badge.svg) | UCRT, Mini-GMP, Threads, JIT, LTO, Gnu Filesystem Hierarchy |
| [Windows Standalone Zip]() | | MinGW-w64 as portable Zip file |
| [Windows MSIX]() | | MinGW-W64 packaged as MSIX |
| [Mac OS](https://github.com/spk121/guile-portability/actions/workflows/macos.yml) | ![Mac OS](https://github.com/spk121/guile-portability/actions/workflows/macos.yml/badge.svg) | |

## Bugs

| Project | ID | Status | Description |
|---------|----|--------|-------------|
| Guile   | 12345 | TODO | N/A |

