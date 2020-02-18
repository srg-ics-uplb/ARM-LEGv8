# Single-Cycle CPU

## Testing in Ubuntu 18.04
```
$ sudo apt install iverilog gtkwave
$ iverilog -o CPU_TEST CPU_TEST.v ARM_CPU.vi
$ vvp -v CPU_TEST -i 
$ gtkwave waveform.vcd 
```




## Objective

This functional single cycle (non-pipelined) processor is capable of performing basic arithmetic, logic and data operations. It is based on the ARM 64-bit architecture, with 32 registers each 64-bits wide with instruction lengths of 32-bits. 

Basic assembly instructions: `LDUR`, `STUR`, `ADD`, `SUB`, `ORR`, `AND`, `CBZ`, `B`, and `NOP` are supported by the CPU, with `LDUR` and `STUR` supporting immediate values when performing certain operations to the registers module.

During development, the group wrote a sample program (in ARMv7 assembly) and tested it on a Cypress PSoC 5LP and recorded its registers after each instruction. The group then verified this custom CPU with the state of the registers from the Cypress PSoC 5LP.

When working with the unconditional branch, instead of a written label the tested instruction use immediate values to where the location of memory where the program counter will branch to. 

Since the ARM CPU is little endian, the instruction memory in this project is designed to have 64 8-bits for each index. 

The Data Memory is made up of 31 64-bit values to show that the values could be accessed and stored via the CPU. 

With the instruction memory, data memory and registers located outside the CPU itself, the project could be incrementally tested and treated as independent components on a physical board. 

## Supported ISA

The examples below use the following 'variables' to show off the functionaility for each instruction:

- `r#`: Register # in the CPU (From 0 to 31)
- `RAM`: Random Access Memory (RAM) or Data Memory
- `PC`: Program Counter

### LDUR: Load RAM into Registers

Example 1: `LDUR r2, [r10]`

- Sudo-C code: `r2 = RAM[r10]`
- Explanation: Retrieve the value in memory at location `r10` and put that value into register `r2`

Example 2: `LDUR r3, [r10, #1]`

- Sudo-C code: `r3 = RAM[r10 + 1]`
- Explanation: Retrieve the value in memory at the location `r10 + immediate (1)` and put that value into register `r3`

### STUR: Store Registers into RAM

Example 1: `STUR r1, [r9]`

- Sudo-C code: `RAM[r9] = r1`
- Explanation: Store the value of `r1` into memory at the location `r9`

Example 2: `STUR r4, [r7, #1]`

- Sudo-C code: `RAM[r7 + 1] = r4`
- Explanation: Store the value of `r4` into memory at the location `r7 + immediate (1)`

### ADD: Add Registers

Example: `ADD r5, r3, r2`

- Sudo-C code: `r5 = r3 + r2`
- Explanation: Add the values of `r3` and `r2` then put the result into `r5`

*Note: This does not support immediate values. Only values within the registers!*

### SUB: Subtract Registers

Example: `SUB r4, r3, r2`

- Sudo-C code: `r4 = r3 - r2`
- Explanation: Subtract the values of `r3` and `r2`  then put the result into `r4`

*Note: This does not support immediate values. Only values within the registers!*

### ORR: Bit-wise OR Registers

Example: `ORR r6, r2, r3`

- Sudo-C code: `r6 = r2 | r3`
- Explanation: Bit-wise OR the values of `r2` and `r3` then put the result into `r6`

*Note: This does not support immediate values. Only values within the registers!*

### AND: Bit-wise AND Registers

Example: `AND r4, r3, r2`

- Sudo-C code: `r4 = r3 & r2`
- Explanation: Bit-wise AND the values of `r3` and `r2` and put the result into `r4`

*Note: This does not support immediate values. Only values within the registers!*

### CBZ: Conditional Jump (when the value in Register is zero)

Example: `CBZ r1, #2`

- Sudo-C code: `if (r1 == 0) { PC = 2 } else { PC++ }`
- Explanation: If the value of `r1` is zero then jump to instruction 2, otherwise, continue executing `PC++`

### B: Unconditional (arbitrary) Jump

Example: `B #2`

- Sudo-C: `PC = 2`
- Explanation: Jump to instruction 2

### NOP: No Operation

Example: `NOP`

- Sudo-C: `;`
- Explanation: An instruction that makes the processor wait one clock cycle.

## Test Program (Instructions)

The thirteen instructions as shown in the table below is the test program used to test the functionality of the CPU.

| Line # |      ARM Assembly     |                Machine Code             | Hexadecimal|
|:------:|:----------------------|:---------------------------------------:|:----------:|
|    1   | `LDUR r2, [r10]`    | 1111 1000 0100 0000 0000 0001 0100 0010 | 0xF8400142 |
|    2   | `LDUR r3, [r10, #1]`| 1111 1000 0100 0000 0001 0001 0100 0011 | 0xF8401143 |
|    3   | `SUB r4, r3, r2`    | 1100 1011 0000 0010 0000 0000 0110 0100 | 0xCB020064 |
|    4   | `ADD r5, r3, r2`    | 1000 1011 0000 0010 0000 0000 0110 0101 | 0x8B020065 |
|    5   | `CBZ r1, #2`        | 1011 0100 0000 0000 0000 0000 0100 0001 | 0xB4000041 |
|    6   | `CBZ r0, #2`        | 1011 0100 0000 0000 0000 0000 0100 0000 | 0xB4000040 |
|    7   | `LDUR r2, [r10]`    | 1111 1000 0100 0000 0000 0001 0100 0010 | 0xF8400142 |
|    8   | `ORR r6, r2, r3`    | 1010 1010 0000 0011 0000 0000 0100 0110 | 0xAA030046 |
|    9   | `AND r7, r2, r3`    | 1000 1010 0000 0011 0000 0000 0100 0111 | 0x8A030047 |
|   10   | `STUR r4, [r7, #1]` | 1111 1000 0000 0000 0001 0000 1110 0100 | 0xF80010E4 |
|   11   | `B #2`              | 0001 0100 0000 0000 0000 0000 0000 0011 | 0x14000003 |
|   12   | `LDUR r3, [r10, #1]`| 1111 1000 0100 0000 0001 0001 0100 0011 | 0xF8401143 |
|   13   | `ADD r8, r0, r1`    | 1000 1011 0000 0001 0000 0000 0000 1000 | 0x8B010008 |

## Test Program (Registers and Data Memory Setup)

The instructions were entered into the instruction memory itself to properly show its functionality while simulated. 

The Registers were initialized with values from 0-30 with register 31 defined set to 0 as stated in the reference sheet for LEGv8. 

The Data Memory was initialized with values starting from 0-3100 with each content of memory being 100 more than the previous index with an exception of index 10 and 11 with the contents 1540 and 2117 respectively.

## Source Directories

- **ARM LEGv8 CPU Module** - ARM_CPU.v

- **ARM LEGv8 Testbench** - CPU_TEST.v
