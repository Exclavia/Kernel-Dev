# Table of contents
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


## Additional Information & Resources
- [Project Files](https://github.com/Exclavia/Kernel-Dev/blob/main/files/)
- [Intel Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [OSdev.org](https://wiki.osdev.org/Expanded_Main_Page)
- [OSdever.net](http://www.osdever.net/tutorials/)
- [alt.os.development](https://groups.google.com/g/alt.os.development)
- [Original Website](http://www.jamesmolloy.co.uk/tutorial_html/)


<br>

___

<p align="center">© 2008 - James Molloy</p>
<p align="center"><i>james@jamesmolloy.co.uk</i></p>
