- fos

```
bootsec.s
loader.c

kentry.s
init.c

=========
-entry:
[bits 16]
[org 0x7c00]

global _start

_start:

==todo stack_setup
    jmp 0:loader


- bydefault realmode: 
-- 1M limition with segmemt register set; 
-- 64k limition with segment register=0.

loader:
==todo read_sects of second part of boot loader(7 sects) to addr e.g 0x10000
==todo move x to new addr?
==todo move gdt to new addr?

    jnc pm_setup:

pm_setup:
==todo enable_a20
==todo lgdt xxx, cr0set
==todo ljmp $CODE_SEL, $sec_boot------------------in gdt, code seg base....
    
    ==below can be done in c code
    jmp CODE_SEG:init_paging

sec_boot:
    ==todo registers.
    ==call boot_main(in this c: load_kernel(from 8 sector, 15M), KER_START())


//////0-7 sectors for boot: 0 bootsector, 1-7 second boot in c code.
//////8->..; around 16M kernel data: kernel_entry.s, init.c

kernel_entry.S:
kernel_start:
 ...lgdt, registers.
 ...ljmp kernelcodeseg, $init


init()
...
...
 init_paging:
==todo pagetable_setup
    jmp vm_starting

vm_starting:


=============

cli

enable a20: prepare for extending more than 1M bytes.
load_drive
detect_mem; via e820

protect_mode_setup; gdt desc table setup to load (access control), and protection enable(PE), jump CODESEG(0x8): init_pm

pagetable_setup

virtual mode(once switched, then virtual memory using paging)


https://github.com/nikhilroxtomar/Simple-Operating-System-from-Scratch/blob/master/boot/boot.asm

To refer 
1- tiny os: https://github.com/chobits/tinyos/blob/b5db0bfc78fbcac4bf69644a0d69feb63ff21213/boot/boot.S
2- skelix os: https://github.com/bitristan/skelix-os/blob/main/02-protected-mode/bootsect.s
3- simple os from scratch: https://github.com/nikhilroxtomar/Simple-Operating-System-from-Scratch/blob/master/boot/boot.asm
4- https://github.com/cfenollosa/os-tutorial/blob/master/07-bootsector-disk/boot_sect_main.asm


```

```
==
boot
load_kernel()

kernel_entry()
jmp to kernel_init

kernel_init()
...video_init()
...mm_init
...int_init
...task_init
...block_init
...fs_init
...keyboard_init
...pit_init
...loadinit
....kload_init

```

- reference
  https://wiki.osdev.org/Main_Page
  
- pseudocode for os
  - skelix-os, tinyos, ArvernOS, osdev etc  

- dpdk framework  

   
- real mode bootstrap
```
move the string character to screen memory address.
```
- protected mode bootstrap
```
move 512 bytes of kernel to designed address
move rest of kernel to designed address + 0x200
enable a20
- load gdt
- enter protected mode by set register CR0 bit 0
- ljmp code seg in gdt for protected mode's code executing (load.s?)
- ?

```
- print() func
```
- video memory 0xb8000 for display the characters in screen.
```
- interrupt/exception/syscall
```
- idt install
- pic install
- 3/= 0
```

result = 3/0
``` 
在x86架构的操作系统中，除以零通常会导致一个硬件异常，即除以零错误（#DE）。当CPU执行除法指令（如DIV或IDIV）且操作数为零时，
CPU会触发一个异常，这个异常会通过中断机制通知操作系统。
在汇编语言中，除以零的中断处理过程通常由操作系统的异常处理例程来处理。这些例程是操作系统内核的一部分，它们负责捕获异常，并采取适当的措施，
比如显示错误消息、记录错误、尝试恢复或终止程序。
以下是一个简化的例子，展示了在x86架构的Linux操作系统中，如何处理除以零的异常：

assembly
复制
section .text
    global _start

_start:
    mov eax, 42          ; 被除数
    mov ebx, 0           ; 除数，为零
    cdq                  ; 扩展符号到edx
    div ebx              ; 尝试除以零，这将触发#DE异常

    ; 程序不会执行到这里，因为上面的除法会触发异常

    mov eax, 1           ; 系统调用号（sys_exit）
    xor ebx, ebx         ; 退出状态码
    int 0x80             ; 触发系统调用，退出程序
在上述代码中，div ebx指令会尝试执行除法操作，但由于ebx是零，这将触发一个#DE异常。在用户模式下，这个异常通常会导致程序崩溃，
操作系统会捕获这个异常，并显示一个错误消息，类似于“段错误”（Segmentation Fault）。

在内核模式下，操作系统提供的异常处理例程会接管。这些例程通常是通过中断描述符表（IDT）来注册的。当除以零的异常发生时，
CPU会查找IDT中对应的条目，并跳转到相应的处理函数。
```

- interrupt handling in HW(CPU)
```
除0异常，cpu会触发中断通知操作系统，从CPU收到中断到中断处理程序之前，cpu做了什么处理，汇编代码阐试:


当CPU执行一个除法指令（如DIV或IDIV）并且除数为零时，CPU会触发一个除以零的异常。
在x86架构中，这通常是一个#DE（Divide Error）异常。
从CPU收到中断到中断处理程序开始执行之前，CPU会进行以下处理：

异常发生：当CPU执行除法指令并且发现除数为零时，它会立即停止当前指令的执行。
保存当前状态：CPU会自动执行以下操作：
Push the current FLAGS register (including the interrupt flag) onto the stack.
Clear the interrupt flag to prevent further interrupts (if not already in an interrupt handler).
Push the current code segment (CS) and the instruction pointer (IP) onto the stack,
which indicates where the exception occurred.
For some exceptions, the error code may also be pushed onto the stack. However, for #DE, no error code is pushed.
查找中断描述符：CPU使用中断向量（在这个情况下是#DE的向量，通常是0）来索引中断描述符表（IDT）。IDT是一个数据结构，
它包含了指向各个中断处理程序的指针。
执行中断处理程序：CPU根据IDT中的描述符，加载新的代码段选择子和偏移量到CS和IP寄存器中，从而跳转到相应的中断处理程序。
以下是一个简化的汇编代码示例，展示了CPU在触发#DE异常后可能执行的操作：

assembly
复制
; 假设这是在执行除法指令之前的CPU状态
; 假设EAX包含被除数，EBX包含除数（为零）

; 执行除法指令，触发#DE异常
div ebx

; 以下是由CPU自动执行的，不是汇编代码
; 但是我们可以用汇编指令来模拟这个过程
====================CPU自动完成begin，CPU完成当前指令后，判断到有中断来或同步的处理异常号触发了，
======================接着就自动保存标志信息寄存器PSR/EFLAGS, CS:IP's push，
按照刚记录的中断或异常号查找IDT/EVL,跳转到ISR handle。============
; 保存FLAGS, CS, IP到栈上
pushf
push cs
push ip

; 清除中断标志
cli

; 查找IDT中的#DE描述符
mov ax, 0       ; #DE的中断向量是0----------#DE(divide error) 中断向量是0，其他的异常(ARM EVT table) 类型号或中断号,
---------------------------------------------CPU自动根据中断号或异常类型号，赋值加载到ax。
mov bx, idt_base ; IDT的基地址
shl ax, 3       ; 每个描述符是8字节，所以左移3位
add bx, ax      ; 计算描述符的地址

; 加载CS和IP
mov ax, [bx+2]  ; 描述符中的代码段选择子
mov cs, ax
mov ip, [bx+0]  ; 描述符中的偏移量

; 跳转到中断处理程序
jmp far [bx]    ; 远跳转到CS:IP
=====================CPU自动完成end======================

; 注意：上面的代码是模拟性的，实际上CPU会自动执行这些操作
在实际的CPU执行过程中，上述步骤是由硬件自动完成的，不需要程序员编写这些具体的汇编指令。
程序员通常只需要编写中断处理程序，并将其安装到IDT中。
在现代操作系统中，如Linux和Windows，这些操作通常是由操作系统的内核来管理的，而不是由用户程序直接处理。

```

- interrupt is aync (trigger by HW), Trap is sync way (trigged by instruction or event), syscall is special's trap (user code trigger).
  
- 中断处理完会返回，异常处理完不会返回，为什么呢? 异常处理后，它的逻辑是不会向中断一样去调用iret，返回之前的next指令执行。
- timer interrupt used in varies  
  https://wiki.osdev.org/APIC_timer  
  
- multitasking
- filesystem
- memory management
- executing program
