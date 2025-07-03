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

### 6.2.2. Page directories/tables
<img src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/page_directory_2.png" >
Possibly you've been tapping on your calculator and have worked out that to generate a table mapping each 4KB page to one 32-bit descriptor over a 4GB address space requires 4MB of memory. Perhaps, perhaps not - but it's true.

4MB may seem like a large overhead, and to be fair, it is. If you have 4GB of physical RAM, it's not much. However, if you are working on a machine that has 16MB of RAM, you've just lost a quarter of your available memory! What we want is something progressive, that will take up an amount of space proportionate to the amount of RAM you have.

Well, we don't have that. But intel did come up with something similar - they use a 2-tier system. The CPU gets told about a page directory, which is a 4KB large table, each entry of which points to a page table. The page table is, again, 4KB large and each entry is a page table entry, described above.

This way, The entire 4GB address space can be covered with the advantage that if a page table has no entries, it can be freed and it's present flag unset in the page directory.

### 6.2.3. Enabling paging
Enabling paging is extremely easy.

1. Copy the location of your page directory into the CR3 register. This must, of course, be the physical address.
2. Set the PG bit in the CR0 register. You can do this by OR-ing with 0x80000000.

## 6.3. Page faults
When a process does something the memory-management unit doesn't like, a page fault interrupt is thrown. Situations that can cause this are (not complete):
- Reading from or writing to an area of memory that is not mapped (page entry/table's 'present' flag is not set)
- The process is in user-mode and tries to write to a read-only page.The process is in user-mode and tries to access a kernel-only page.
- The page table entry is corrupted - the reserved bits have been overwritten.

The page fault interrupt is number 14, and looking at [chapter 3](https://github.com/Exclavia/Kernel-Dev/blob/main/03-screen.md) we can see that this throws an error code. This error code gives us quite a bit of information about what happened.

**Bit 0**
> If set, the fault was not because the page wasn't present. If unset, the page wasn't present.

**Bit 1**
> If set, the operation that caused the fault was a write, else it was a read.

**Bit 2**
> If set, the processor was running in user-mode when it was interrupted. Else, it was running in kernel-mode.

**Bit 3**
> If set, the fault was caused by reserved bits being overwritten.

**Bit 4**
> If set, the fault occurred during an instruction fetch.

The processor also gives us another piece of information - the address that caused the fault. This is located in the CR2 register. Beware that if your page fault hander itself causes another page fault exception this register will be overwritten - so save it early!

## 6.4. Putting it into practice
We're almost ready to start implementing. We will, however, need a few assistant functions first, the most important of which are memory management functions.

### 6.4.1. Simple memory management with placement malloc
If you come from a C++ background, you may have heard of 'placement new'. This is a version of new that takes a parameter. Instead of calling malloc, as it normally would, it creates the object at the address specified. We are going to use a very similar concept.

When the kernel is sufficiently booted, we will have a kernel heap active and operational. The way we code heaps, though, usually requires that virtual memory is enabled. So we need a simple alternative to allocate memory before the heap is active.

As we're allocating quite early on in the kernel bootup, we can make the assumption that nothing that is kmalloc()'d will ever need to be kfree()'d. This simplifies things greatly. We can just have a pointer (placement address) to some free memory that we pass back to the requestee then increment. Thus:
```c
u32int kmalloc(u32int sz)
{
  u32int tmp = placement_address;
  placement_address += sz;
  return tmp;
}
```
That will actually suffice. However, we have another requirement. When we allocate page tables and directories, they must be page-aligned. So we can build that in:
```c
u32int kmalloc(u32int sz, int align)
{
  if (align == 1 && (placement_address & 0xFFFFF000)) // If the address is not already page-aligned
  {
    // Align it.
    placement_address &= 0xFFFFF000;
    placement_address += 0x1000;
  }
  u32int tmp = placement_address;
  placement_address += sz;
  return tmp;
}
```

