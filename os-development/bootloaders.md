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

