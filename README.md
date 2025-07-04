# Kernel Development Tutorials
A clone of **James Molloy's** Kernel Development tutorials, grabbed via WayBack archive circa. 2021-2023 (As of July 2025 the website seems to be down.)
- [Original Website](http://www.jamesmolloy.co.uk/tutorial_html/)
- [Additional Notes](#additional-notes)
- [Project Files](https://github.com/Exclavia/Kernel-Dev/blob/main/files/)

## Roll your own toy UNIX-clone OS
This set of tutorials aims to take you through programming a simple UNIX-clone operating system for the x86 architecture. The tutorial uses C as the language of choice, with liberally mixed in bits of assembler. The aim is to talk you through the design and implementation decisions in making an operating system. The OS we make is monolithic in design (drivers are loaded through kernel-mode modules as opposed to user-mode programs), as this is simpler.

This set of tutorials is very practical in nature. The theory is given in every section, but the majority of the tutorial deals with getting dirty and implementing the abstract ideas and mechanisms discussed everywhere. It is important to note that the kernel implemented is a teaching kernel. I know that the algorithms used are not the most space efficient or optimal. They normally are chosen for their simplicity and ease of understanding.The aim of this is to get you into the correct mindset, and to give you a grounding upon which you can work. The kernel given is extensible, and good algorithms can easily be plugged in.If you have problems with the theory, there are plenty of sites that would be delighted to help you (most questions on OSDev forums are concerned with implementation - "My gets function doesn't work! help!" - A theory question is a breath of fresh air to many ;) ). Links can be found at the bottom of the page.

### Table of contents
1. [Environment setup](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/01-environment-setup.md)
2. [Genesis](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/02-genesis.md)
3. [The Screen](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/03-screen.md)
4. [The GDT and IDT](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/04-gdt-and-idt.md)
5. [IRQs and the PIT](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/05-irq-and-pit.md)
6. [Paging](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/06-paging.md)
7. [The Heap](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/07-heap.md)
8. [The VFS and the initrd](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/08-vfs-and-initrd.md)
9. [Multitasking](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/09-multitasking.md)
10. [User Mode](https://github.com/Exclavia/Kernel-Dev/blob/main/chapters/10-user-mode.md)

### Prerequisites
To compile and run the sample code I provide requires just GCC, ld, NASM and GNU Make. NASM is an open-source x86 assembler, and is the assembler-of-choice for many x86 OS-devs.

There is no point, however, in just compiling and running without comprehension. You must understand what is being coded, and as such you should have a very strong knowledge of C, especially regarding pointers. You should also know a little bit of assembly (Intel syntax is used in these tutorials), including what the EBP register is used for.

### Resources
There are plenty of resources out there if you know where to look. In particular, you should find these useful:

- [RTFM!](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) The intel manuals are a godsend
- [OSdev.org](https://wiki.osdev.org/Expanded_Main_Page) wiki and forums.
- [OSdever.net](http://www.osdever.net/tutorials/) has many good tutorials and papers, and in particular Bran's kernel development tutorials, which some of the earlier code from this tutorial is based off. (I myself used these tutorials to get started, and the code is so good I haven't had to change it over the years)
- [alt.os.development](https://groups.google.com/g/alt.os.development) can answer many of your non-n00b questions. N00b questions are better asked on the [osdever.net forums.](http://forums.osdever.net/)

#### Copyright - James Molloy - 2008
***james@jamesmolloy.co.uk***

## Additional Notes
This repository is solely for the purpose of preserving these tutorials so that future developers may use them. **I do not claim to be the owner, nor author of these tutorials.** That solely goes to **James Molloy**. If they, James Molloy, do not want this repository publicized, it is within their full rights to request that I takedown the repository. I will comply with any takedown requests made by James Molloy. *Any questions or requests related to this repository can be sent via messages on Github or via email:* ***git@exclavia.network***

### Changes Made
The repository has some slight changes to it that may differ it from the original tutorial. These are purely due to not being able to find original files, or for clarity. Nothing in the actual tutorials themselves are changed. All original images are shown, as well as all original source code files included at the end of each chapter.
