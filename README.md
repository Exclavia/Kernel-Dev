# Kernel Development Tutorials
An archive of James Molloy's Kernel Development tutorials. As of time of archiving the original website seems to be offline or inaccessible. I used [Wayback Machine](http://web.archive.org/) snapshots from 2021-2023 in order to backup the tutorial pages and, some of, the original tutorial resources/files. I have converted each of the chapters into formatted markdown pages to be easily viewed.

- [Original Website](http://www.jamesmolloy.co.uk/tutorial_html/)
- [About this repository](#about-this-repository)
- [Project Files](/files/)

> [!NOTE]
> **Known bugs & issues**
> 
> This tutorial is known for some bugs and issues, you can read more about in on the [OSDev wiki page dedicated to the tutorial](https://wiki.osdev.org/James_Molloy%27s_Tutorial_Known_Bugs).

## Roll your own toy UNIX-clone OS
This set of tutorials aims to take you through programming a simple UNIX-clone operating system for the x86 architecture. The tutorial uses C as the language of choice, with liberally mixed in bits of assembler. The aim is to talk you through the design and implementation decisions in making an operating system. The OS we make is monolithic in design (drivers are loaded through kernel-mode modules as opposed to user-mode programs), as this is simpler.

This set of tutorials is very practical in nature. The theory is given in every section, but the majority of the tutorial deals with getting dirty and implementing the abstract ideas and mechanisms discussed everywhere. It is important to note that the kernel implemented is a teaching kernel. I know that the algorithms used are not the most space efficient or optimal. They normally are chosen for their simplicity and ease of understanding.The aim of this is to get you into the correct mindset, and to give you a grounding upon which you can work. The kernel given is extensible, and good algorithms can easily be plugged in.If you have problems with the theory, there are plenty of sites that would be delighted to help you (most questions on OSDev forums are concerned with implementation - "My gets function doesn't work! help!" - A theory question is a breath of fresh air to many ;) ). Links can be found at the bottom of the page.

### Table of contents
1. [Environment setup](/chapters/01-environment-setup.md)
2. [Genesis](/chapters/02-genesis.md)
3. [The Screen](/chapters/03-screen.md)
4. [The GDT and IDT](/chapters/04-gdt-and-idt.md)
5. [IRQs and the PIT](/chapters/05-irq-and-pit.md)
6. [Paging](/chapters/06-paging.md)
7. [The Heap](/chapters/07-heap.md)
8. [The VFS and the initrd](/chapters/08-vfs-and-initrd.md)
9. [Multitasking](/chapters/09-multitasking.md)
10. [User Mode](/chapters/10-user-mode.md)


### Prerequisites
To compile and run the sample code I provide requires just [GCC](https://gcc.gnu.org/), [ld](https://www.gnu.org/software/binutils/), [NASM](https://www.nasm.us/) and [GNU Make](https://www.gnu.org/software/make/). NASM is an open-source x86 assembler, and is the assembler-of-choice for many x86 OS-devs.

There is no point, however, in just compiling and running without comprehension. You must understand what is being coded, and as such you should have a very strong knowledge of C, especially regarding pointers. You should also know a little bit of assembly (Intel syntax is used in these tutorials), including what the EBP register is used for.

### Resources
There are plenty of resources out there if you know where to look. In particular, you should find these useful:

- [Intel Arch Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [OSdev.org](https://wiki.osdev.org/Expanded_Main_Page) wiki and forums.
- [OSdever.net](http://www.osdever.net/tutorials/) has many good tutorials and papers, and in particular Bran's kernel development tutorials, which some of the earlier code from this tutorial is based off. (I myself used these tutorials to get started, and the code is so good I haven't had to change it over the years)
- [alt.os.development](https://groups.google.com/g/alt.os.development) can answer many of your non-n00b questions. N00b questions are better asked on the [osdever.net forums.](http://forums.osdever.net/)

<br>

___

<p align="center">© 2008 - James Molloy</p>


## About This Repository 
This repository exists solely to **preserve and archive** the original set of tutorials authored by **James Molloy**, which were previously available on his [personal website](http://jamesmolloy.co.uk). As of the time of archiving, the original source appears to be offline or inaccessible. These tutorials have historically been a valuable resource for developers, especially those interested in low-level programming and operating system development.

I would like to express **full credit and authorship to James Molloy** for the creation of this work. I make **no claim of ownership** over the material contained within this repository. This archive has been made available under the [Creative Commons CC0 license](/LICENSE.md), in the spirit of open access and educational preservation.

If **James Molloy**, as the original author, wishes for this repository to be modified, removed, or taken down for any reason, I will **comply fully and immediately** with any such request.

For questions, feedback, or takedown requests, I can be reached via:
- [GitHub messages](https://github.com/Exclavia) (or [issues](https://github.com/Exclavia/Kernel-Dev/issues/new))
- Email: [git@exclavia.network](mailto:git@exclavia.network)



### Changes made / Submitting errors
The repository has some slight changes to it that may differ it from the original tutorial. These are purely due to not being able to find original files, or for clarity. Nothing in the actual tutorials themselves are changed. All original images are shown, as well as all original source code files included at the end of each chapter.

If you happen to notice something that doesn't make sense, or if you have found some errors, please let me know by [making a new issue](https://github.com/Exclavia/Kernel-Dev/issues/new). 
