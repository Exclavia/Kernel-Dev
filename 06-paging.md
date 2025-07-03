# 6. Paging
In this chapter we're going to enable paging. Paging serves a twofold purpose - memory protection, and virtual memory (the two being almost inextricably interlinked).

## 6.1. Virtual memory (theory)
If you already know what virtual memory is, you can skip this section.

In linux, if you create a tiny test program such as:
```c
int main(char argc, char **argv)
{
  return 0;
}
```
compile it, then run 'objdump -f', you might find something similar to this.
```
jamesmol@aubergine:~/test> objdump -f a.out

a.out:     file format elf32-i386
architecture: i386, flags 0x00000112:
EXEC_P, HAS_SYMS, D_PAGED
start address 0x080482a0
```
Notice the start address of the program is at 0x80482a0, which is about 128MB into the address space. It may seem strange, then, that this program will run perfectly on machines with < 128MB of RAM.

What the program is actually 'seeing', when it reads and writes memory, is a virtual address space. Parts of the virtual address space are mapped to physical memory, and parts are unmapped. If you try to access an unmapped part, the processor raises a page fault, the operating system catches it, and in POSIX systems delivers a SIGSEGV signal closely followed by SIGKILL.

This abstraction is extremely useful. It means that compilers can produce a program that relies on the code being at an exact location in memory, every time it is run. With virtual memory, the process thinks it is at, for example, 0x080482a0, but actually it could be at physical memory location 0x1000000. Not only that, but processes cannot accidentally (or deliberately) trample other processes' data or code.

Virtual memory of this type is wholly dependent on hardware support. It cannot be emulated by software. Luckily, the x86 has just such a thing. It's called the MMU (memory management unit), and it handles all memory mappings due to segmentation and paging, forming a layer between the CPU and memory (actually, it's part of the CPU, but that's just an implementation detail).

## 6.2. Paging as a concretion of virtual memory
Virtual memory is an abstract principle. As such it requires concretion through some system/algorithm. Both segmentation (see [chapter 3](https://github.com/Exclavia/Kernel-Dev/blob/main/03-screen.md)) and paging are valid methods for implementing virtual memory. As mentioned in chapter 3 however, segmentation is becoming obsolete. Paging is the newer, better alternative for the x86 architecture.

Paging works by splitting the virtual address space into blocks called pages, which are usually 4KB in size. Pages can then be mapped on to frames - equally sized blocks of physical memory.

### 6.2.1. Page entries
Each process normally has a different set of page mappings, so that virtual memory spaces are independent of each other.
<img src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/paging_pte_1.png" >
In the x86 architecture (32-bit) pages are fixed at 4KB in size. Each page has a corresponding descriptor word, which tells the processor which frame it is mapped to. Note that because pages and frames must be aligned on 4KB boundaries (4KB being 0x1000 bytes), the least significant 12 bits of the 32-bit word are always zero. The architecture takes advantage of this by using them to store information about the page, such as whether it is present, whether it is kernel-mode or user-mode etc. The layout of this word is in the picture.

The fields in that picture are pretty simple, so let's quickly go through them.

**P**
> Set if the page is present in memory.

**R/W**
> If set, that page is writeable. If unset, the page is read-only. This does not apply when code is running in kernel-mode (unless a flag in CR0 is set).

**U/S**
> If set, this is a user-mode page. Else it is a supervisor (kernel)-mode page. User-mode code cannot write to or read from kernel-mode pages.

**Reserved**
> These are used by the CPU internally and cannot be trampled.

**A**
> Set if the page has been accessed (Gets set by the CPU).

**D**
> Set if the page has been written to (dirty).

**AVAIL**
> These 3 bits are unused and available for kernel-use.

**Page frame address**
> The high 20 bits of the frame address in physical memory.


