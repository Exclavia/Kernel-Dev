# Kernel Development Tutorials

> [About this repository](#about-this-repository)

An archive of [James Molloy's Kernel Development tutorials](http://www.jamesmolloy.co.uk/tutorial_html/). As of the time of archiving, the original website seems to be offline or inaccessible.

I used [Wayback Machine](http://web.archive.org/) snapshots from 2021-2023 in order to backup the tutorial pages and, some of, the original tutorial resources/files. Each chapter has been converted into formatted markdown pages to be easily viewed.

If you happen to notice something that doesn't make sense, or if you have found an error, please let me know by [making a new issue](https://github.com/Exclavia/Kernel-Dev/issues/new).

</br>

> [!NOTE]
> **Known bugs & issues**
> 
> This tutorial is known for some bugs and issues, you can read more about in on the [OSDev wiki page dedicated to the tutorial](https://wiki.osdev.org/James_Molloy%27s_Tutorial_Known_Bugs).

</br>

## Roll your own toy UNIX-clone OS
> This set of tutorials aims to take you through programming a simple UNIX-clone operating system for the x86 architecture. The tutorial uses C as the language of choice, with liberally mixed in bits of assembler. The aim is to talk you through the design and implementation decisions in making an operating system. The OS we make is monolithic in design (drivers are loaded through kernel-mode modules as opposed to user-mode programs), as this is simpler.</p>

> This set of tutorials is very practical in nature. The theory is given in every section, but the majority of the tutorial deals with getting dirty and implementing the abstract ideas and mechanisms discussed everywhere. It is important to note that the kernel implemented is a teaching kernel. I know that the algorithms used are not the most space efficient or optimal. They normally are chosen for their simplicity and ease of understanding. The aim of this is to get you into the correct mindset, and to give you a grounding upon which you can work. The kernel given is extensible, and good algorithms can easily be plugged in.</p>

> If you have problems with the theory, there are plenty of sites that would be delighted to help you (most questions on OSDev forums are concerned with implementation - "My gets function doesn't work! help!" - A theory question is a breath of fresh air to many ;) ). Links can be found at the bottom of the page.</p>

</br>

## Prerequisites
Tools needed to compile and build the code within the tutorials: 
 - [Netwide Assembler (NASM)](https://www.nasm.us/)
 - [GNU Compiler (GCC)](https://gcc.gnu.org/)
 - [GNU Binutils (ld)](https://www.gnu.org/software/binutils/)
 - [GNU Make](https://www.gnu.org/software/make/)


> There is no point, however, in just compiling and running without comprehension. You must understand what is being coded, and as such you should have a very strong knowledge of C, especially regarding pointers. You should also know a little bit of assembly (Intel syntax is used in these tutorials), including what the EBP register is used for.

</br>

## Table of contents
1. [Environment setup](/chapters/01-environment-setup.md)

    - 1.1. [Directory Structure](/chapters/01-environment-setup.md#11-directory-structure)
    - 1.2. [Compiling](/chapters/01-environment-setup.md#12-compiling)
    - 1.3. [Running](/chapters/01-environment-setup.md#13-running)
    - 1.4. [Bochs](/chapters/01-environment-setup.md#14-bochs)
    - 1.5. [Useful Scripts](/chapters/01-environment-setup.md#15-useful-scripts)
2. [Genesis](/chapters/02-genesis.md)

    - 2.1. [The boot code](/chapters/02-genesis.md#21-the-boot-code)
    - 2.2. [Understanding the boot code](/chapters/02-genesis.md#22-understanding-the-boot-code)
    - 2.3. [Adding some C code](/chapters/02-genesis.md#23-adding-some-c-code)
    - 2.4. [Compiling, linking and running!](/chapters/02-genesis.md#24-compiling-linking-and-running)
3. [The Screen](/chapters/03-screen.md)

    - 3.1. [The theory](/chapters/03-screen.md#31-the-theory)
    - 3.2. [The practice](/chapters/03-screen.md#32-the-practice)
    - 3.3. [Summary](/chapters/03-screen.md#33-summary)
    - 3.4. [Extensions](/chapters/03-screen.md#34-extensions)
4. [The GDT and IDT](/chapters/04-gdt-and-idt.md)

    - 4.1. [The Global Descriptor Table (theory)](/chapters/04-gdt-and-idt.md#41-the-global-descriptor-table-theory)
    - 4.2. [The Global Descriptor Table (practical)](/chapters/04-gdt-and-idt.md#42-the-global-descriptor-table-practical)
    - 4.3. [The Interrupt Descriptor Table (theory)](/chapters/04-gdt-and-idt.md#43-the-interrupt-descriptor-table-theory)
    - 4.4. [The Interrupt Descriptor Table (practical)](/chapters/04-gdt-and-idt.md#44-the-interrupt-descriptor-table-practical)
5. [IRQs and the PIT](/chapters/05-irq-and-pit.md)

    - 5.1. [Interrupt requests (theory)](/chapters/05-irq-and-pit.md#51-interrupt-requests-theory)
    - 5.2. [Interrupt requests (practical)](/chapters/05-irq-and-pit.md#52-interrupt-requests-practical)
    - 5.3. [The PIT (theory)](/chapters/05-irq-and-pit.md#53-the-pit-theory)
    - 5.4. [The PIT (practical)](/chapters/05-irq-and-pit.md#54-the-pit-practical)
6. [Paging](/chapters/06-paging.md)

    - 6.1. [Virtual memory (theory)](/chapters/06-paging.md#61-virtual-memory-theory)
    - 6.2. [Paging as a concretion of virtual memory](/chapters/06-paging.md#62-paging-as-a-concretion-of-virtual-memory)
    - 6.3. [Page faults](/chapters/06-paging.md#63-page-faults)
    - 6.4. [Putting it into practice](/chapters/06-paging.md#64-putting-it-into-practice)
7. [The Heap](/chapters/07-heap.md)

    - 7.1. [Data structure description](/chapters/07-heap.md#71-data-structure-description)
    - 7.2. [Algorithm description](/chapters/07-heap.md#72-algorithm-description)
    - 7.3. [Implementing an ordered list](/chapters/07-heap.md#73-implementing-an-ordered-list)
    - 7.4. [The heap itself](/chapters/07-heap.md#74-the-heap-itself)
    - 7.5. [Testing](/chapters/07-heap.md#75-testing)
    - 7.6. [Summary](/chapters/07-heap.md#76-summary)
8. [The VFS and the initrd](/chapters/08-vfs-and-initrd.md)

    - 8.1. [The virtual filesystem](/chapters/08-vfs-and-initrd.md#81-the-virtual-filesystem)
    - 8.2. [The initial ramdisk](/chapters/08-vfs-and-initrd.md#82-the-initial-ramdisk)
    - 8.3. [My own solution](/chapters/08-vfs-and-initrd.md#83-my-own-solution)
    - 8.4. [Loading the initrd as a multiboot module](/chapters/08-vfs-and-initrd.md#84-loading-the-initrd-as-a-multiboot-module)
    - 8.5. [Testing it out](/chapters/08-vfs-and-initrd.md#85-testing-it-out)
9. [Multitasking](/chapters/09-multitasking.md)

    - 9.1. [Tasking theory](/chapters/09-multitasking.md#91-tasking-theory)
    - 9.2. [Cloning an address space](/chapters/09-multitasking.md#92-cloning-an-address-space)
    - 9.3. [Creating a new stack](/chapters/09-multitasking.md#93-creating-a-new-stack)
    - 9.4. [Actual multitasking code](/chapters/09-multitasking.md#94-actual-multitasking-code)
    - 9.5. [Testing](/chapters/09-multitasking.md#95-testing)
    - 9.6. [Summary](/chapters/09-multitasking.md#96-summary)
10. [User Mode](/chapters/10-user-mode.md)

    - 10.1. [Switching to user mode](/chapters/10-user-mode.md#101-switching-to-user-mode)
    - 10.2. [System calls](/chapters/10-user-mode.md#102-system-calls)
    - 10.3. [Testing](/chapters/10-user-mode.md#103-testing)

</br>

## Resources
- [OSdev.org Wiki & Forums](https://wiki.osdev.org/Expanded_Main_Page)
- [OSdever Tutorials & Forums](http://www.osdever.net/tutorials/)
- [BrokenThorn OS Development Series](http://www.brokenthorn.com/Resources/OSDevIndex.html)
- [The Little OS Book](https://littleosbook.github.io/)
- [Intel Software Development Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [MikeOS Project](https://mikeos.sourceforge.net/)
- [FusionOS Project](https://0xc0ffee.netlify.app/osdev/)

</br>

## About This Repository 
This repository exists solely to **preserve and archive** the original set of tutorials authored by **James Molloy**, which were previously available on his [personal website](http://jamesmolloy.co.uk). As of the time of archiving, the original source appears to be offline or inaccessible. These tutorials have historically been a valuable resource for developers, especially those interested in low-level programming and operating system development.

I would like to express **full credit and authorship to James Molloy** for the creation of this work. I make **no claim of ownership** over the material contained within this repository. This archive has been made available under the [Creative Commons CC0 license](/LICENSE.md), in the spirit of open access and educational preservation.

If **James Molloy**, as the original author, wishes for this repository to be modified, removed, or taken down for any reason, I will **comply fully and immediately** with any such request.

For questions, feedback, or takedown requests, I can be reached via:
- GitHub: [Issue](https://github.com/Exclavia/Kernel-Dev/issues/new)
- E-mail: [git@exclavia.network](mailto:git@exclavia.network)

</br>

 ---

<p align="center">&copy; <b>2008 James Molloy - james@jamesmolloy.co.uk</b></p>
<p align="center"><i>Archived: July 2025 by Tristan Wehler</i></p>

---
