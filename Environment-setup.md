# 1. Environment setup
We need a base from which to design and make our kernel. Here I will be assuming that you are using a *nix system, with the GNU toolchain. If you want to use a windows system, you must either use cygwin (which is a *nix emulation environment) or DJGPP. Either way, the makefiles and commands in this tutorial may not work.

## 1.1. Directory structure
My directory is laid out as follows:
```
tutorial
 |
 +-- src
 |
 +-- docs
```
All your source files will go in src, and all your documentation (you do write documentation? ;) ) should go in docs.

## 1.2. Compiling
The examples in this tutorial should compile successfully with the GNU toolchain (gcc, ld, gas, etc). The assembly examples are written in intel syntax, which is (my personal opinion) a much more human-readable syntax than the AT&T syntax that GNU AS uses. To assemble these, you will need the [Netwide Assembler](https://www.nasm.us/).

This tutorial is not a bootloader tutorial. We will be using [GRUB](https://www.gnu.org/software/grub/) to load our kernel. To do this, we need a floppy disk image with GRUB preloaded onto it. There are tutorials to do this, but, happily, I have made a standard image, which can be found here. This goes in your 'tutorial' (or whatever you named it) directory.

