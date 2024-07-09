
- qemu
```
qemu-system-i386 -drive file=kernel.bin,format=raw -nographic

qemu-system-x86_64 -drive file=kernel.bin,format=raw -nographic
```


- mmap paging


```
https://github.com/povilasb/simple-os/blob/master/kernel/src/paging.cc


//visus puslapius nustatyti i not present
void init_paging()
{
    int i;

    for (i = 0; i < FRAMES_COUNT; i++)
        frames[i] = 0; //unused

    //frames for page directory and page tables are set as in use
    frame_setUsage(PAGE_DIRECTORY_START, 1);
    for (i = 0; i < PAGE_TABLE_COUNT; i++)
        frame_setUsage(PAGE_TABLES_START + i, 1);

    pageDirectory = (PageDirectory*) frame_address(PAGE_DIRECTORY_START);
    //all pages are set as not present
    for (i = 0; i < 1024; i++)
    {
        set_pageTableEntry(&pageDirectory->entry[i], 0, 0, 0, 0); //not present
        // PageTable* pageTable = (PageTable*) frame_address(PAGE_TABLES_START + i);
        //for (j = 0; j < 1024; j++)
            //set_pageTableEntry(&pageTable->entry[j], 0, 0, 0, 0); //not present
    }

    set_pageDirectory(pageDirectory);

    //map kernel source code where virtual addr = physical addr
    for (i = 0; i < KERNEL_SOURCE_SIZE; i++)
    {
        map_page(pageDirectory, KERNEL_START_ADDR + i*FRAME_SIZE, KERNEL_START_ADDR + i*FRAME_SIZE);
    }

    //map kernel stack where virtual addr = physical addr
    for (i = 0; i < 4; i++)
        map_page(pageDirectory, KERNEL_STACK_START_ADDR + i*FRAME_SIZE, KERNEL_STACK_START_ADDR + i*FRAME_SIZE);
    map_page(pageDirectory, VIDEO_MEM_START, 0xB8000); //mapping virtual video memory

    //mapping kernel heap where virtual addr = physical addr
    for (i = 0; i < KERNEL_HEAP_SIZE; i++)
        map_page(pageDirectory, KERNEL_HEAP_START_ADDR + i*FRAME_SIZE, KERNEL_HEAP_START_ADDR + i*FRAME_SIZE);

    //map page directory and page tables
    map_page(pageDirectory, frame_address(PAGE_DIRECTORY_START), frame_address(PAGE_DIRECTORY_START));
    for (i = 0; i < PAGE_TABLE_COUNT; i++)
        map_page(pageDirectory, frame_address(PAGE_TABLES_START + i), frame_address(PAGE_TABLES_START + i));

    paging_enable();
}
```

paging 
https://wiki.osdev.org/Paging
```
U/S, the 'User/Supervisor' bit, controls access to the page based on privilege level.
If the bit is set, then the page may be accessed by all;
if the bit is not set, however, only the supervisor can access it. For a page directory entry, the user bit controls access to all the pages referenced by the page directory entry. Therefore if you wish to make a page a user page, you must set the user bit in the relevant page directory entry as well as the page table entry.

```
