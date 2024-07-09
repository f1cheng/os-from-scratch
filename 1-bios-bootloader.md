1. bios loads the bootloader into fixed memroy address 0x7c00. 
> why fixed, as boot.bin doesn't have header structure field to indicate the load address (probably ELF file has the header structure).
2. bootloader's 'org 0x7c00' is used for compile to convert far address with offset 0x7c00. 
> https://stackoverflow.com/questions/67393581/why-do-we-need-org-0x7c00-at-the-beginning-of-a-bootloader 

