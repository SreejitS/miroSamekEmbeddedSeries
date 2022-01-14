# Embedded Systems Course Notes

This document will have the notes for [this](https://www.youtube.com/playlist?list=PLGGUgnOTqaTvNkh003iPRPZ6sEtq7DRR4) playlist.

## Lesson 1: Counting

- Mental image: Big table of bytes, also we using C99 dialect(have to look more into this)
- Fundamental Definitions:
  - Memory = Just a big table of bytes
  - Variables = Just a location in computer memory to store value ie. bytes ie. instructions
  - Address = Simply numbers assigned to these locations
  - Instructions = Content of these locations

- In hexadecimal `0x0` to `0xA` is represented using 4-bits. So to represent a byte you'll need 2 hex digits. 32-bits have 4-bytes, so need 8 hex digits.
- Most of the ARM Cortex instruction takes 2 bytes in memory and their purpose is to manipulate(store/clear/add etc.) contents inside the registers.
- ARM Cortex has 16 32-bit wide general purpose R0-R15. R15 is PC. PC stores the address of the instruction to execute.
- Instructions can typically manipulate registers in one clock cycle.
- Signed = integers are stored as 2's complement and can represent positive as well as negative numbers(basically integers)
- `0x0000 0000`(0) ... `0x7fff ffff`(2^31-1 or 2147483647) (largest positive value 32-bit signed integer can represent).
- On adding 1 to that, we make it to `0x8000 0000`(-2^31 or -2147483648) (smallest negative value 32-bit signed integer can represent) all the way to `0xffff ffff`(largest negative value it can represent ie -1).
- On incrementing this again we get to `0x0000 0000` and the cycle repeats.
- For unsigned integers it starts from `0x0000 0000`(smallest positive:0) to `0xffff ffff` (largest positive:`4294967295`)

[Saving-IAR-windows-layout](https://stackoverflow.com/questions/38459556/iar-window-layout)

## Lesson 2: Flow of Control

- Typically, in pure sequential or linear code, the PC increments as a side affect after executing instructions to point to the next instruction. We use flow control to affect this hard wired behavior so as to have decision making at run time.
- This will allow to loop(ie. avoid repetitions) or skip specific parts of the code(ie. to make decisions).
- Basically set of instruction that manipulate PC register directly.
- Generally the info about how much to jump ie change the PC is encoded as a part of the instructions themselves.
- B instruction is a branch instruction that modifies the PC so it skips over a few Instructions.
- CMP instruction modifies APSR(Application Program Status Register) - N/Z/C/V/Q flags
- Conditional branch BLT.N jumps on negative flag set. The op code has the info about the offset to be added to the current PC, so can jump up or down the code. For ex. 0x7e:0xdbfc - 0xd means encoding T1/0xfc is the offset ie. add current value of PC to offset to know the address to jump 0x7e+0xfc(signed) so 0x7e-0x04 = 0x7a. The PC jumps backwards therefore we have a loop.
- Effects of loop in time critical code(ex. interrupt processing):
  - **Loop overhead**: the processor has to execute additional tests and jumps to handle the loop. Therefore can use loop unrolling to speed things up in time critical codes.
  - **Pipeline stalls**: ARM Cortex M uses pipeline to increase the throughput and increase the number of instructions processed in given time. Processor works with multiple instructions at various stages of completion. One instruction is divided into steps like fetch from memory,decode and execute. These individual steps take one clock cycle to execute. Pipeline works at full capacity when instructions are executed in order. When the ordering is disrupted by branch instruction, it has to discard the partially processed Instructions and restart pipelining at the new branched address as it has to now fetch the instructions pointed by the new value of the PC.

## Lesson 3: Variables and Pointers

- Mental image: one location has value, which happen to be another location in the big table of bytes.
- The whole point of having instructions is to do operations on the data stored in registers. This is due to ARM being a
RISC architecture processor.
- To store value to and to load value from, we need the knowledge of the addresses. So they are very important. So important, that in C we have variables that store only addresses, called pointers.
- ARM has RISC architecture, therefore memory can only be read by special load instruction and all the data manipulations must
happen in the data registers, and the modified registers values can be stored back in the memory by store instruction as opposed to CISC. STR and LDR.N involves memory addresses.
- Instead of writing something like p_int=0x20000000 directly, C has mechanism to store raw addresses in pointers by type casting. So, compiler will accept, p_int = (int *)0x20000000U; Note, this type casting is for the value stored in that address.

```C
int *p_int;
/* Best way to handle C declarations is to read it backwards,
so p_int, is a variable(location in the big table of bytes) 
that stores address(location) and those locations have
integer values.
 */
```

- referencing is to get the address, de-referencing is to get to the integer value stored there.
`p_int = (int *)0x20000002u;`//misaligned address can overwrite existing values in memory addresses `0x20000000`
- As the address `0x2000 0000` has `0x2000 0000`, `0x2000 0001`, `0x2000 0002`, `0x2000 0003` to store a 32 bit integer(4 bytes) and then uses `0x2000 0004` ... `0x2000 0007` to store the next integer. **p_int** is intentionally misaligned and code will store whatever value to `0x2000 0002` corrupting a stored integer if already in placed at `0x2000  0000`

## Lesson 4: Blinking the LED

- Mental image: The big table of bytes are divided into sections, for flash, ram, peripherals, SFR's.
- To blink LED, you have to know about the memory map of the microcontroller.
- The technique to blocking clock to certain parts of the chip to save power is called clock gating.
- So initially you cannot read any data at address of PORTF in the debugger.
- While in debug mode, on setting the bit on clock gating control register, it provides clock to PORTF and the debugger can see the data in that address range. The hardware block wakes up.
- Controlling hardware boils down to writing a number to a memory address. So use pointers in C and deference the address directly to access the hardware and put a value to it.

```C
//to provide clock to PORTF
*((unsigned int *)0x400FE608U) = 0x20U; 
/* Note on type casting: the raw address is pre-appended by the the
type of the value as in how to interpret the raw value in front of it.
& operator does not need this as the compiler understands it from the
variable declaration
*/
```

- We can make it blink by adding to looping statements after turning on and off the LED.
