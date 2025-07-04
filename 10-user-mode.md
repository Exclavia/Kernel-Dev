# 10. User mode (and syscalls)
Your kernel, at the moment, is running with the processor in "kernel mode", or "supervisor mode". Kernel mode makes available certain instructions that would usually be denied a user program - like being able to disable interrupts, or halt the processor.
<img src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/rings.png" >
Once you start running user programs, you'll want to make the jump from kernel mode to user mode, to restrict what instructions are available. You can also restrict read or write access to areas of memory. This is often used to 'hide' the kernel's code and data from user programs.

## 10.1. Switching to user mode
The x86 is strange in that there is no direct way to switch to user mode. The only way one can reach user mode is to return from an exception that began in user mode. The only method of getting there in the first place is to set up the stack as if an exception in user mode had occurred, then executing an exception return instruction (IRET).

<img align="right" src="https://raw.githubusercontent.com/Exclavia/Kernel-Dev/refs/heads/main/assets/iret.png">

The IRET instruction expects, when executed, the stack to have the following contents (starting from the stack pointer - the lowermost address upwards):
 - The instruction to continue execution at - the value of EIP.
 - The code segment selector to change to.The value of the EFLAGS register to load.
 - The stack pointer to load.
 - The stack segment selector to change to.
