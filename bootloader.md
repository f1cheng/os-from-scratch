
boot loader

```
bootsec.s
loader.c

kentry.s
init.c

=====bootsec.s
-entry:
[bits 16]
[org 0x7c00]

global _start

;first 512bytes(stage1) is loaded already by BIOS: start=0x7C00, size=0x200; so [0x7C00-0x7E00) in RAM <-> sector1 in disk
_start:
boot_stage1:
    stack_setup---when this is neccessary, put it there. HAHA: it is when push and pop used. implictly in call func.
    enable_a20; ready for above 1M bytes operation
    load_stage2_from_driver
    jmp boot_state2


- bydefault realmode: 
-- 1M limition with segmemt register set; 
-- 64k limition with segment register=0.

==todo read_sects of second part of boot loader(7 sects) to addr e.g 0x10000
==todo move x to new addr?
==todo move gdt to new addr?

.org 508
.word 0x0000	/* for sects of kernel image */
.word 0xaa55

boot_stage2:;---------------; -start=0x7E00, size=0x200 x 7(sectors), so [0x7e00-0x8c00) in RAM <->sector2-8 in disk
    jnc pm_setup: or call pm_setup


;e820 entries stored in [0x8c00,  0x9800), other params: [0x9800, 0x10000)---up to 65k max
pm_setup:
    load gdt
    turn_on_pm_mode
    reset_stack
    call stage2_main


=====loader.c
stage2_main:
    load_kernel()
    (void(*) KERNEL_START)()



/* segment base is 0x7c00 */
#define BOOT_PADDR(offset) ((offset) + 0x7c00)!!!!!!!!!!!!!!!
#define KERNEL_BASE	0xc0000000
#define PADDR(va)	((va) - KERNEL_BASE)
#define VADDR(pa)	((pa) + KERNEL_BASE)
KLINK	= -T kernel.ld

kernel.elf:$(OBJS)
	$(Q)$(LD) $(KLINK) $^ -o $@

	/* text section: start at 0xc1000000 */
	. = 0xc1000000;!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!all lables address in kernel has this offset added!!!!!
=========>!!!!! code label address needs convert to Physical address by minus 0xc000000.

=====kentry.s;----------------------kernel data to [0x1000000 - 0x1e00000] = assume max 14M bytes for kernel image [16M-30M]
====================================0x01000000=16M = KERNEL_START
kernel_start:
    ljmp kernelcodeseg, $init

=====init.c
    init() -->page_init() ;vm used after that.

page_init():
    pagetable_setup()
    enable_paging()



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

0x1e00000-above: kern_page_dir, vm = 0xc1e00000

=============
boot_alloc()
allocated real space from 0x1e00000-above, but ret with VADR(ret)

kern_page_dir 
kern_page_table 

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
