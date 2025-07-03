# 5. IRQs and the PIT

In this chapter we're going to be learning about interrupt requests (IRQs) and the programmable interval timer (PIT).

## 5.1. Interrupt requests (theory)

There are several methods for communicating with external devices. Two of the most useful and popular are polling and interrupting.

**Polling**
> Spin in a loop, occasionally checking if the device is ready.

**Interrupts**
> Do lots of useful stuff. When the device is ready it will cause a CPU interrupt, causing your handler to be run.

As can probably be gleaned from my biased descriptions, interrupting is considered better for many situations. Polling has lots of uses - some CPUs may not have an interrupt mechanism, or you may have many devices, or maybe you just need to check so infrequently that it's not worth the hassle of interrupts. Any rate, interrupts are a very useful method of hardware communication. They are used by the keyboard when keys are pressed, and also by the programmable interval timer (PIT).

<img src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/pics.png" >

The low-level concepts behind external interrupts are not very complex. All devices that are interrupt-capable have a line connecting them to the PIC (programmable interrupt controller). The PIC is the only device that is directly connected to the CPU's interrupt pin. It is used as a multiplexer, and has the ability to prioritise between interrupting devices. It is, essentially, a glorified 8-1 multiplexer. At some point, someone somewhere realised that 8 IRQ lines just wasn't enough, and they daisy-chained another 8-1 PIC beside the original. So in all modern PCs, you have 2 PICs, the master and the slave, serving a total of 15 interruptable devices (one line is used to signal the slave PIC).

The other clever thing about the PIC is that you can change the interrupt number it delivers for each IRQ line. This is referred to as remapping the PIC and is actually extremely useful. When the computer boots, the default interrupt mappings are:
- IRQ 0..7 - INT 0x8..0xF
- IRQ 8..15 - INT 0x70..0x77

This causes us somewhat of a problem. The master's IRQ mappings (0x8-0xF) conflict with the interrupt numbers used by the CPU to signal exceptions and faults (see last [chapter](https://github.com/Exclavia/Kernel-Dev/blob/main/04-gdt-and-idt.md)). The normal thing to do is to remap the PICs so that IRQs 0..15 correspond to ISRs 32..47 (31 being the last CPU-used ISR).

