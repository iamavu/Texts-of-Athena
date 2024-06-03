# Bootloader

## <mark style="color:purple;">Boot Process</mark>

### <mark style="color:red;">Power On</mark>

When power button is pressed it sends out a electric signal to motherboard which is then re-routed to PSU.&#x20;

It contains single bit of data i.e. current state which can be either 0 or 1. After PSU receives this signal it starts supplying appropriate amount of current to every part and then once that is done, it sends a signal to BIOS stating everything is good.

### <mark style="color:red;">POST</mark>

Once the signal is received from PSU, BIOS starts something called POST (Power On Self Test) which insures that good amount of power is being supplied to parts and for any memory corruption bugs in RAM

Then the control is shifted from POST to BIOS via POST placing a jump instruction to the BIOS and processor starts executing it

### <mark style="color:red;">BIOS</mark>

BIOS creates IVT (Interrupt Vector Table), provides interrupt services, ensures that there are no hardware problems and supplies setup utility

BIOS tries to find OS by executing a Interrupt (INT) `0x19` to find a bootable device, in case it doesn't find that, it will return - "No Operating System Found" and halt the system

#### <mark style="color:green;">IVT (Interrupt Vector Table)</mark>

Interrupts can be executed by different programs, they are loaded at the address of `0x00`.&#x20;

If there is switch in processor modes, the interrupts won't be available; neither hardware or software.

### <mark style="color:red;">BIOS INT 0x19</mark>

It warm reboots the system without clearing memory or restoring the IVT and it reads the first sector (Sector 1, Head 0, Track 0)

* Sector is group of 512 bytes
* Head is side of the disk, front or back
* Track is collection of sectors

## <mark style="color:purple;">Theory</mark>

### <mark style="color:red;">Hardware Exceptions</mark>

These are executed by processor and similar to software exceptions

### <mark style="color:red;">CLI and STI</mark>&#x20;

These can be used to enable or disable interrupts, all of them. `cli` clears them and `sti` enables them

### <mark style="color:red;">Fault Hardware Exceptions</mark>

* Double Fault - If processor finds invalid instruction (such as division by zero) it executes interrupt `0x8` which is second fault exception handler
* Triple Fault - If processor can't continue after double fault, it executes triple fault i.e. it hard reboots

## <mark style="color:purple;">Bootloader.asm</mark>

{% code title="bootloader.asm" lineNumbers="true" fullWidth="false" %}
```nasm
org 0x7c00
bits 16

start:
  
    cli 
    hlt

times 510 - ($ - $$) db 0
dw 0xaa55
```
{% endcode %}

* Line 1 : We are loaded by BIOS at the address of `0x7c00` and all addresses after this are relative to 0x7c00
* Line 2 : x86 family is backwards compatible hence all x86 computers boot in 16bit mode
* Line 4 to 7 : Label consisting of clearing all interrupts and then halting the system
* Line 9 : The `$` represents address at the current line and `$$` represents address of the first instruction, so `$-$$` returns numbers of bytes from current line to the start and as boot signature should be last two bytes of the bootsector so we use `times` keyword to fill the rest bytes with 0
* Line 10 : As [#bios](bootloader.md#bios "mention") looks for a bootable disk and it identifies that with boot signature which is 0xAA55&#x20;

### <mark style="color:red;">Assembling The Bootloader</mark>

Assemble the `bootloader.asm` using NASM with following command : `nasm -f bin bootloader.asm -o bootloader.bin` and then run it using `qemu-system-x86_64 -drive format=raw,file=bootloader.bin`

## <mark style="color:purple;">Processor Modes</mark>

### <mark style="color:red;">Real Mode</mark>

* Uses `segment:offset` memory model
* Is limited 1MB(+64k) of memory
* No virtual memory or memory protection

#### <mark style="color:green;">Segment:Offset Memory Mode</mark>

Segment is section of memory. `CS, DS, ES and SS` store the base addresses of segment

* CS (Code Segment) - Stores base segment address for code
* DS (Data Segment) - Stores base segment address for data
* ES (Extra Segment) - Stores base segment address for anything
* SS (Stack Segment) - Stores base segment address for the stack

Offset is a number added to the base i.e. `offset = base number + offset number`

So now the formula becomes `Absolute (exact) Memory Address = (Segment Address * 16) + Offset` (16 decimal multiplication was done so as to address each segment in real mode is 16bits)

Multiple `segment:offset` pairs can point to same memory location. If we have two segments within 64KB, the segments can overlap causing memory corruption, hence the real mode doesn't have memory protection

### <mark style="color:red;">Protected Mode (PMode)</mark>

* 32 bit processor mode; uses 32 bit registers and access up to 4GB RAM
* Uses descriptor table allowing memory protection

### <mark style="color:red;">Unreal Mode</mark>

* Real mode with address space of protected mode
* To enable simply switch from real mode to protected mode and then back to real mode after loading the new descriptor

### <mark style="color:red;">Virtual 8086 Mode (v86 Mode)</mark>

* Represents protected mode in 16bit real mode emulated environment
* Allows use of BIOS interrupts within PMode as they are only available in real mode

## <mark style="color:purple;">Expanding Bootloader</mark>

<pre class="language-nasm" data-line-numbers><code class="lang-nasm">org     0x7c00  
bits    16 

start: 
    jmp loader

;*************************************************;
;	 OEM Parameter Block
;*************************************************;

times 0Bh-$+start DB 0

bpbBytesPerSector:  	DW 512
bpbSectorsPerCluster: 	DB 1
bpbReservedSectors: 	DW 1
bpbNumberOfFATs: 	DB 2
bpbRootEntries: 	DW 224
bpbTotalSectors: 	DW 2880
bpbMedia: 	        DB 0xF0
bpbSectorsPerFAT: 	DW 9
bpbSectorsPerTrack: 	DW 18
bpbHeadsPerCylinder: 	DW 2
bpbHiddenSectors: 	DD 0
bpbTotalSectorsBig:     DD 0
bsDriveNumber: 	        DB 0
bsUnused: 	        DB 0
bsExtBootSignature: 	DB 0x29
bsSerialNumber:	        DD 0xa0a1a2a3
bsVolumeLabel: 	        DB "MOS FLOPPY "
bsFileSystem: 	        DB "FAT12   "

<strong>msg	db	"welcome to dumOS", 0
</strong><strong>
</strong>Print:
    lodsb  
    or      al, al 
    jz      PrintDone 
    mov     ah, 0x0e
    int     0x10 
    jmp     Print 

PrintDone:
    ret 

;*************************************************;
;	Bootloader Entry Point
;*************************************************;

loader:
    xor     ax, ax 
    mov     ds, ax 
    mov     es, ax 
    
    mov     si, msg 
    call    Print 
    
    cli
    hlt

times 510 - ($ - $$) db 0 
dw  0xaa55
</code></pre>

* Line 32 : Message that we have to print being terminated by null
* Line 35 to 40 : Load each byte into `al` and print it using teletype (`0x0e`) until it reaches null (0)
* Line 50 to 55 : Set segment registers to 0 as we have org as `0x7c00` hence all addresses are based from `0x7c00:0` so we nullify them
* Line 54 to 55 : Set source index to `msg` and then call the `Print` function

## <mark style="color:purple;">Rings of Assembly</mark>&#x20;

Rings in assembly represent level of protection and control the program has over system. There are four rings - Ring 0, Ring 1, Ring 2, Ring 3

Smaller the ring number, the more control and protection the software has i.e Ring 0 programs have absolute control over anything in the system.

## <mark style="color:purple;">Multi Stage Bootloader</mark>

### <mark style="color:red;">Single Stage Bootloader</mark>

Bootloader/BootSector are only 512 bytes in size, if the bootloader executes kernel directly within that same space, it is called as single stage bootloader but as this is very difficult to do so, most bootloaders are multi stage bootloaders

### <mark style="color:red;">Multi Stage Bootloader</mark>

Multi stage bootloader executes another bootloader (usually 16bit) which then executes the 32 bit kernel

## Loading Sectors Off Disk

Stage 2 bootloader is the one which we will be using to load our kernel - The Kernel Loader

