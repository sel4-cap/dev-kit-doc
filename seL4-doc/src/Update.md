# Picolibc

Microkit does not contain a bundled libc; therefore, it was decided to port picolibc to Microkit. Picolibc is a C library specifically designed for use in embedded systems, providing a lightweight and efficient implementation of the C standard library functions. This makes it an ideal choice to complement Microkit, enhancing its capabilities without adding unnecessary overhead.

The ported picolibc for Microkit is linked [here] https://github.com/sel4-cap/picolibc/tree/main with instructions for building and linking with Microkit. To use dynamic memory allocation (malloc) the __heap_start and __heap_end symbols (defined in the system file) needed to be added to the picosbrk.c file. This tells picolibc where the memory location of the heap.

# U-boot

U-boot has been ported to be used with microkit (https://github.com/sel4-cap/u-boot/commits/microkit). The majority of the changes were made to remove name space clashes with picolibc.

# Device tree

Inline assembly is used to import the device tree to microkit:

```c
#define STR2(x) #x
#define STR(x) STR2(x)
#define INCBIN_SECTION ".rodata"
#define INCBIN(name, file) \
    __asm__(".section " INCBIN_SECTION "\n" \
            ".global incbin_" STR(name) "_start\n" \
            ".balign 16\n" \
            "incbin_" STR(name) "_start:\n" \
            ".incbin \"" file "\"\n" \
            \
            ".global incbin_" STR(name) "_end\n" \
            ".balign 1\n" \
            "incbin_" STR(name) "_end:\n" \
            ".byte 0\n" \
    ); \
    extern __attribute__((aligned(16))) const char incbin_ ## name ## _start[]; \
    extern                              const char incbin_ ## name ## _end[] 
INCBIN(device_tree, DTB_PATH); 

const char* _end = incbin_device_tree_end;
```

# Wrapper code 

To set up the uboot drivers, the CAmkES code has to extract address and size information of each node in the device tree and map the device resources into the virtual address space. This is handed in the uboot_drivers.c file. However, in Microkit the mapping of the physical devices and the mapping of physical to virtual addresses is handled in the system file therefore this code can be removed from the uboot_drivers.c file. Furthermore, there was no dma library for microkit therefore the CAmkES dma library was adapted to be used with Microkit. The dma microkit dma library is stored in this repository (https://github.com/sel4-cap/libmicrokitdma).

# Logging

- All U-Boot logging routines onto `UBOOT_LOG` routines at an equivalent logging level.



