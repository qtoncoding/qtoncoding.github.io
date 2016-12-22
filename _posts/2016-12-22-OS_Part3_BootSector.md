BIOS and boot sector
====================

Now that we have the tools we need, let’s get down to business!

First up is how does the BIOS know where our bootloader code is to run it? Turns
out, what is does is to look at the first sector on a drive (512 bytes), look at
offset (location) 510 and 511, and if it sees the “magic” numbers 0x55 and 0xAA,
it will know that that sector is the boot sector, load it onto RAM and run the
code there.

Let’s create a .asm file, call it boot\_sect.asm and fill it with the following:

{% highlight assembly %}

; Pad the remaining bytes in this sector with 0

times 510 - ($-$$) db 0

; magic number for BIOS to recognize us as a boot sector

dw 0xaa55

{% endhighlight %}

In the code above, the ```time``` directive is a NASM specific directive,
which is practically “repeat this x times”. (alternatively you can use
```dup```, which is basically the same thing)

```$``` signifies the current instruction location, and ```$$```
is the start of this segment. So ```$-$$``` means the distance from the
top to the current instruction. ```510 – ($-$$)``` is the remaining
bytes that aren’t used from 510 total bytes.

```db 0``` is a “Data directive” that tells the assembler to put a byte of
value 0 to that location.

Combined, we have: put a byte of value 0 into every remaining location of the
first 510 bytes.

Next up is ```dw 0xaa55```. This again is a data directive, telling the
assembler to put a word with value 0xaa55 at this location. Previously we filled
up to the 509th byte with 0, and at 510 and 511, we put 0xAA and 0x55. This is
the “magic number” that will let the BIOS know this is a boot sector.

Assembler, assemble!
====================

Now that we have the assembly code, we want the assembler to make this into
machine code. To do this, we use NASM with this configuration:

```nasm boot_sect.asm -f bin -o boot_sect.bin```

This will use NASM to assemble the ```boot_sect.asm``` file, outputting
binary format (just the machine code) with ```-f bin``` and write the
output to a file with ```-o boot_sect.bin```.

Emulator, run!
==============

Now that we have the code, let’s try run it! On Windows, use QEmu with this:

```qemu-system-i386.exe -fda boot_sect.bin```

This is ask QEmu to emulate the i386 architecture and use the
```boot_sect.bin``` file as the first Floppy drive.

You should see nothing after QEmu saying it’s booting from Floppy Drive, and
that’s OK because we didn’t do anything in our boot sector. Next up, we’ll do
something that you can verify on the screen.

Interrupt and BIOS
==================

Before we continue, there is an important concept to understand, that is
“interrupt”. An interrupt generally is a signal to the processor from a hardware
or software to signify that it requires attention. The CPU then suspend whatever
it’s doing (hence, interrupt), saving the states and perform a function called
“interrupt handler”. After the handler function exits, the processor returns to
whatever it was doing before. More information on interrupt can be found here.

The BIOS uses interrupt as a way to, in a sense, invoke functions that it
provides to the bootloader. The usage is to set the high bits of the A register
(AH) to some value to determine what function you want the BIOS to perform, set
the lower bits of the A register (AL) to the value you want the function to use,
and raise the corresponding interrupt using the instruction INT with the
interrupt code as argument.

The BIOS provides some basic display functionality, and we’ll use this mechanism
to display a character on the screen.

Print a character to the screen.
================================

The interrupt we’ll use is 0x10, and the code to put in register AH is 0x0E, and
we’ll print character ‘X’ (feel free to use whatever ASCII character you want).
Here is the code:

{% highlight assembly %}

; Set BIOS routine to printing a character

mov ah, 0x0E

; Set the character to print

mov al, 'X'

; Raise interupt

int 0x10

{% endhighlight %}

In this code, we put the value ```0x0E``` into register ```ah```
with the instruction ```mov```. We similarly put the value ‘X’ into
register ```al```. Finally, we raise interrupt ```0x10``` with
instruction ```int```.

Put this code before the code from the section above, assemble it with NASM and
use QEmu to run the resulting file, and you should see an X on the top left of
the screen. Congratulations, we’ve successfully booted from a disk drive and
even print something on the screen to confirm it.

Next time we’ll write some useful functions for our bootloader to use, like
printing a null terminated string, printing a hex number and loading memory from
disk.

Cheers,

Thai
