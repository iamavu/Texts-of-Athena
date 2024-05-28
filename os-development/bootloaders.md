# Bootloaders

## <mark style="color:purple;">Boot Process</mark>

### <mark style="color:red;">Power On</mark>

When power button is pressed it sends out a electric signal to motherboard which is then re-routed to PSU.&#x20;

It contains single bit of data i.e. current state which can be either 0 or 1. After PSU receives this signal it starts supplying appropriate amount of current to every part and then once that is done, it sends a signal to BIOS stating everything is good.

### <mark style="color:red;">POST</mark>

Once the signal is received from PSU, BIOS starts something called POST (Power On Self Test) which insures that good amount of power is being supplied to parts and for any memory corruption bugs in RAM

Then the control is shifted from POST to BIOS via POST placing a jump instruction to the BIOS and processor starts executing it

### <mark style="color:red;">BIOS</mark>

BIOS creates IVT (Interrupt Vector Table), provides interrupt services, ensures that there are no hardware problems and supplies setup utility

BIOS tries to find OS by executing a Interrupt (INT) 0x19 to find a bootable device, in case it doesn't find that, it will return - "No Operating System Found" and halt the system

#### <mark style="color:green;">IVT (Interrupt Vector Table)</mark>

Interrupts can be executed by different programs, they are loaded at the address of 0x00.&#x20;

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

* Double Fault - If processor finds invalid instruction (such as division by zero) it executes interrupt 0x8 which is second fault exception handler
* Triple Fault - If processor can't continue after double fault, it executes triple fault i.e. it hard reboots

## <mark style="color:purple;">Bootloader.asm</mark>

<pre class="language-nasm" data-title="bootloader.asm" data-line-numbers data-full-width="false"><code class="lang-nasm">org 0x7c00
bits 16

start:
  
    cli 
<strong>    hlt
</strong><strong>
</strong><strong>times 510 - ($ - $$) db 0
</strong><strong>dw 0xaa55
</strong></code></pre>

* Line 1 : We are loaded by BIOS at the address of 0x7c00 and all addresses after this are relative to 0x7c00
* Line 2 : x86 family is backwards compatible hence all x86 computers boot in 16bit mode
* Line 4 to 7 : Label consisting of clearing all interrupts and then halting the system
* Line 9 : The `$` represents address at the current line and `$$` represents address of the first instruction, so `$-$$` returns numbers of bytes from current line to the start and as boot signature should be last two bytes of the bootsector so we use `times` keyword to fill the rest bytes with 0
* Line 10 : As [#bios](bootloaders.md#bios "mention") looks for a bootable disk and it identifies that with boot signature which is 0xAA55&#x20;
