# Polaris Processor Core

## Description

Polaris is the first member of a family of processors conforming to the Redtail Architecture.  It aims to solve a broad range of small systems and peripheral control problems at minimum cost to the user.  It implements the RISC-V RV64I and RV64S instruction sets.

The Polaris specification includes by reference several external specifications that determines RISC-V instruction set compatibility and/or hardware interoperability in the open-hardware community:
 
- Where this document references the *user specification* or *user spec*, please refer to [The RISC-V Instruction Set Manual, Volume I: User-Level ISA, Version 2.0](http://riscv.wpengine.com/specifications/).

- Where this document references the *supervisor specification* or *supervisor spec*, please refer to [The Draft RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Version 1.7](http://riscv.wpengine.com/specifications/privileged-isa/).

- Where this document references the *Wishbone specification* or *Wishbone spec*, please refer to the [Wishbone B4: WISHBONE System-on-Chip (SOC) Interconnection Architecture for Portable IP Cores](http://cdn.opencores.org/downloads/wbspec_b4.pdf) specification.

- Where this document references the *SDB specification* or *SDB spec*, please refer to the [Self-Describing Bus V1.1](http://www.ohwr.org/projects/sdb/wiki) specification.

## Features

- Wishbone B3 and B4 compatible front-side buses.
- Separate instruction and data bus interfaces permits use in Harvard-architecture designs.
- Optional Bus Interface Unit (BIU) permits use in Von Neumann architectures. 
- 64-bit data and address widths.
- Eight byte select lines enables memory to be addressed at byte granularity.
- 61 instructions.
- 31 integer registers (X0 hardwired to zero).
- 19 CPU Specific Registers (CSRs).
- Four external interrupts.
- Built-in wall-clock, cycle, and retired instruction counters to support profiling.
- Most instructions execute in a single clock cycle.
- Multiple bus master support.
- Pipelined architecture.
- 64-bit general purpose output port.
- 64-bit general purpose input port.

## Block Diagram

      +---------------------------------------------------------- RESET
      | +-------------------------------------------------------- CLK
      | | +------------------------------------------------------ IRQ
      | | | /===================================================< INP
      | | | || /================================================> OUT
      v v v \/ ||
    +-------------+      Pipeline:
    |             |     +-------------+     +---------------+
    | Sequencing  |<--->| Inst. Fetch |<===>|               |
    | and Control |     +-------------+     |               |
    |             |<--->| Decode      |     |               |
    |      +------+     +-------------+     |               |
    |      |      |<===>| Reg. Fetch  |     | Bus Interface |
    |      |      |     +-------------+     |      Unit     |<===> Wishbone master bus
    |      |      |<--->| Operate     |     |               |
    |      | Regs |     +-------------+     |               |
    |      |      |<--->| Data R/W    |<===>|               |
    |      |      |     +-------------+     |               |
    |      |      |<===>| Writeback   |     |               |
    +------+------+     +-------------+     +---------------+


## Programming Model

    Integer Registers       CPU Specific Registers

    X0 (always 0)           UCYCLE    (R/O)
    X1                      UTIME     (R/O)
    X2                      UINSTRET  (R/O)
    X3                      
    X4                      MCPUID    (R/O)
    X5                      MIMPID    (R/O)
    X6                      MHARTID   (R/O)
    X7                      Mstatus
    X8                      MTVEC
    X9                      MTDELEG   (R/O)
    X10                     MIE
    X11                     MTIMECMP
    X12                     MTIME
    X13                     MSCRATCH
    X14                     MEPC
    X15                     MCAUSE
    X16                     MBADADDR
    X17                     MIP
    X18                     MTOHOST
    X19                     MFROMHOST (R/O)
    X20                     
    X21                     
    X22                     
    X23                     
    X24                     
    X25                     
    X26                     
    X27                     
    X28                     
    X29                     
    X30                     
    X31                     

    PC

## Instruction Set Reference

Except where noted, all instructions execute with one clock period of latency.

The following notation is used:

| Notation | Meaning |
|:--------:|:--------|
|csr|A 12-bit, unsigned, integer constant specifying one of 4096 CPU-Specific Registers.|
|disp13|A 13-bit, sign-extended displacement from the current instruction's address.  Bit 0 is ignored, letting the assembler pack the value into a 12-bit field.|
|disp21|A 21-bit, sign-extended displacement from the current instruction's address.  Bit 0 is ignored, letting the assembler pack the value into a 20-bit field.|
|imm5|A 5-bit, unsigned, integer constant used to specify an immediate operand for CSR manipulation.|
|imm6|A 6-bit, unsigned, integer constant used to specify how far to shift the contents of an integer register left or right.|
|imm12|A 12-bit, sign-extended, integer constant ("immediate value").|
|imm32|A 32-bit, sign-extended, integer constant.|
|Xn|A number between 0 and 31 inclusive, indicating an integer register X0 to X31.|

### ADD, SUB, SLL, SLT, SLTU, XOR, SRL, SRA, OR, AND

    ADDI Xd, Xa, imm12
    SLTI Xd, Xa, imm12
    SLTIU Xd, Xa, imm12
    XORI Xd, Xa, imm12
    ORI Xd, Xa, imm12
    ANDI Xd, Xa, imm12
    SLLI Xd, Xa, imm6
    SRLI Xd, Xa, imm6
    SRAI Xd, Xa, imm6

These instructions compute a full-width integer eponymously-named function given a set of source registers.  The result is assigned to register Xd.

#### ADD, AND, OR, SLT, SLTU, SLL, SRL, XOR

|31 |30 |29 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:-:|:-:|:------:|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|0|0|Xb|Xa|fn3|Xd|01100|11|

|fn3|Syntax|Operation|
|:-:|:----:|:-------|
|000|ADD|REG(Xd) := REG(Xa) + REG(Xb)|
|001|SLL|REG(Xd) := LSH(REG(Xa), REG(Xb) AND 63)|
|010|SLT|REG(Xd) := REG(Xa) < REG(Xb)  (signed comparison)|
|011|SLTU|REG(Xd) := REG(Xa) < REG(Xb)  (unsigned comparison)|
|100|XOR|REG(Xd) := REG(Xa) XOR REG(Xb)|
|101|SRL|REG(Xd) := RSH(REG(Xa), REG(Xb) AND 63)
|110|OR|REG(Xd) := REG(Xa) OR REG(Xb)|
|111|AND|REG(Xd) := REG(Xa) AND REG(Xb)|

##### ADD

This instruction sets Xd to the sum of the values in Xa and Xb.

##### AND, OR, XOR

These instructions computes the logical-AND, logical-OR, or logical-XOR functions between the value in Xa and Xb.

##### SLT, SLTU

These instructions set Xd to either 1 or 0, depending on whether or not the value in Xa is less than Xb.  Note that SLTU uses an unsigned comparison.

#### SLL, SRL

These instructions shift the value in Xa left (SLL) or right (SRL) by an amount determined by the lowest six bits in Xb.  All other bits in Xb are ignored.  These are *logical* shifts.

#### SRA, SUB

|31 |30 |29..25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:-:|:-:|:----:|:------:|:------:|:-----:|:----:|:-:|
|0|1|0|Xb|Xa|fn3|Xd|01100|11|

|fn3|Syntax|Operation|
|:-:|:----:|:-------|
|000|SUB|REG(Xd) := REG(Xa) - REG(Xb)|
|101|SRA|REG(Xd) := ASR(REG(Xa), REG(Xb) AND 63)|

##### SRA

This instruction shifts the value in Xa right by an amount determined by the lowest six bits of Xb.  All other bits in Xb are ignored.  This is an *arithmetic* shift, meaning sign is preserved.

##### SUB

This instruction sets Xd to the difference between the values in Xa and Xb.

### ADDI, ANDI, ORI, SLLI, SLTI, SLTIU, SRAI, SRLI, XORI

    ADDI Xd, Xa, imm12
    SLTI Xd, Xa, imm12
    SLTIU Xd, Xa, imm12
    XORI Xd, Xa, imm12
    ORI Xd, Xa, imm12
    ANDI Xd, Xa, imm12
    SLLI Xd, Xa, imm6
    SRLI Xd, Xa, imm6
    SRAI Xd, Xa, imm6

These instructions compute a full-width integer eponymously-named function given a source register and a sign-extended 12-bit immediate value, or an unsigned 6-bit immediate value.  The result is assigned to register Xd.

#### ADDI, ANDI, ORI, SLTI, SLTIU, XORI

|31 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|imm12|Xa|fn3|Xd|00100|11|

|fn3|Syntax|Operation|
|:-:|:----:|:-------|
|000|ADDI|REG(Xd) := REG(Xa) + SX(imm12)|
|010|SLTI|REG(Xd) := REG(Xa) < SX(imm12)  (signed comparison)|
|011|SLTIU|REG(Xd) := REG(Xa) < SX(imm12)  (unsigned comparison)|
|100|XORI|REG(Xd) := REG(Xa) XOR SX(imm12)|
|110|ORI|REG(Xd) := REG(Xa) OR SX(imm12)|
|111|ANDI|REG(Xd) := REG(Xa) AND SX(imm12)|

##### ADDI

This instruction sets Xd to the sum of the values in Xa and the sign-extended imm12 field.

##### ANDI, ORI, XORI

These instructions computes the logical-AND, logical-OR, or logical-XOR functions between the value in Xa and the provided, sign-extended 12-bit immediate value.

##### SLTI, SLTIU

These instructions set Xd to either 1 or 0, depending on whether or not the value in Xa is less than the sign-extended, 12-bit immediate constant provided.  Note that SLTIU uses an unsigned comparison.

#### SLLI, SRAI, SRLI

|31 |30 |29..26|25 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:-:|:-:|:----:|:------:|:------:|:-----:|:----:|:-:|
|0|A|0|imm6|Xa|fn3|Xd|00100|11|

These instructions shift the value in Xa left (SLLI) or right (SRAI, SRLI), respectively.  The value in imm6 determines the number of bit positions moved.  The A flag is 0 for logical shift, 1 for arithmetic.

The A flag **must** be 0 for all left shifts.

|fn3|Syntax|Operation|
|:-:|:----:|:-------|
|001|SLLI|REG(Xd) := LSH(REG(Xa), imm6)|
|101|SRLI, SRAI|REG(Xd) := (IF instruction[62]=1 THEN RSH ELSE ASR)(REG(Xa), imm6)|

### ADDIW, SLLIW, SRLIW, SRAIW

    ADDIW Xd, Xa, imm12
    SLLIW Xd, Xa, imm5
    SRLIW Xd, Xa, imm5
    SRAIW Xd, Xa, imm5

These instructions perform as their ADDI, SLLI, SRLI, and SRAI counter-parts, but only consider the low 32-bits of their source operands.  Additionally, when setting the contents of Xd, the upper 32-bits contain a sign-extension of the lower 32-bits.

#### ADDIW

|31 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|imm12|Xa|000|Xd|00110|11|

Adds a sign-extended immediate constant to the low 32-bits of register Xa.  After sign-extending the sum, store in Xd.

    REG(Xd) := REG(Xa) + SX(imm12)
    REG(Xd)[63:32] := 32{REG(Xd)[31]}


Note that `ADDIW Xd, Xa, 0` can be used to sign-extend a 32-bit quantity in Xa.

#### SLLIW

|31 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|imm5|Xa|000|Xd|00110|11|

Shifts the lower 32-bits of Xa left by the specified number of bits.  After sign-extending the result, store in Xd.

    REG(Xd) := LSH(REG(Xa)[31:0], imm5)
    REG(Xd)[63:32] := 32{REG(Xd)[31]}

#### SRAIW, SRLIW

|31 |30 |29 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|A|0|imm5|Xa|000|Xd|00110|11|

|A|Syntax|Function|
|:-:|:----:|:-------|
|0|SRLIW|REG(Xd) = RSH(REG(Xa)[31:0], imm5)|
|1|SRAIW|REG(Xd) = {32{r[31]}, r}, where r = ASR(REG(Xa)[31:0], imm5)|

Shifts the lower 32-bits of Xa right by the specified number of bits.  After sign-extending the result, store in Xd.

### ADDW, SUBW, SLLW, SRLW, SRAW

    ADDW Xd, Xa, Xb
    SLLW Xd, Xa, Xb
    SRLW Xd, Xa, Xb
    SRAW Xd, Xa, Xb
    SUBW Xd, Xa, Xb

These instructions perform as their ADD, SLL, SRL, SRA, and SUB counter-parts, but only consider the low 32-bits of their source operands.  Additionally, when setting the contents of Xd, the upper 32-bits contain a sign-extension of the lower 32-bits.

#### ADDW, SUBW

|31 |30 |29 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:-:|:-:|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|A|0|Xb|Xa|000|Xd|01110|11|

Adds or subtracts Xb to/from Xa, putting the result in Xd.  The source values are treated as 32-bit quantities.  The result is sign-extended to 64-bits before storing in Xd.

|A|Syntax|Function|
|:-:|:---:|:----|
|0|ADDW|REG(Xd) := {32{r[31], r}}, where r = REG(Xa)[31:0] + REG(Xb)[31:0]|
|1|SUBW|REG(Xd) := {32{r[31], r}}, where r = REG(Xa)[31:0] - REG(Xb)[31:0]|

#### SLLW

|31 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|Xb|Xa|000|Xd|01110|11|

Shifts the lower 32-bits of Xa left by the number of bits specified in Xb[4:0].  After sign-extending the result, store in Xd.

    REG(Xd) := LSH(REG(Xa)[31:0], REG(Xb) AND 31)
    REG(Xd)[63:32] := 32{REG(Xd)[31]}

#### SRAW, SRLW

|31 |30 |29 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|A|0|Xb|Xa|000|Xd|01110|11|

|A|Syntax|Function|
|:-:|:----:|:-------|
|0|SRLW|REG(Xd) = RSH(REG(Xa)[31:0], REG(Xb) AND 31)|
|1|SRAW|REG(Xd) = {32{r[31]}, r}, where r = ASR(REG(Xa)[31:0], REG(Xb) AND 31)|

Shifts the lower 32-bits of Xa right by the specified number of bits.  After sign-extending the result, store in Xd.

### AUIPC Xd, imm32

|31 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:-----:|:----:|:-:|
|imm20   |Xd     |00101 | 11|

    REG(Xd) := PC + (SX(imm32) AND $FFFFFFFFFFFFF000)

### B

    BEQ Xa, Xb, disp13
    BGE Xa, Xb, disp13
    BGEU Xa, Xb, disp13
    BLT Xa, Xb, disp13
    BLTU Xa, Xb, disp13
    BNE Xa, Xb, disp13

|31 |30 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 8|7  |6 .. 2|1 0|
|:-:|:------:|:------:|:------:|:------:|:-----:|:-:|:----:|:-:|
|disp[12]|disp[10:5]|Xb|Xa|cc |disp[4:1]|disp[11]|11000|11|

If the indicated condition is true with respect to the provided registers, jump to a new location in the program, determined by the sum of the branch instruction's address and the `disp13` parameter.

|cc |Syntax|COND(Xa, Xb) = TRUE IF|
|:-:|:----:|:-------|
|000|BEQ   |Xa = Xb|
|001|BNE   |Xa <> Xb|
|100|BLT   |Xa < Xb (signed)|
|101|BGE   |Xa >= Xb (signed)|
|110|BLTU  |Xa < Xb (unsigned)|
|111|BGEU  |Xa >= Xb (unsigned)|

    IF cc = 2 OR cc = 3 THEN
        TRAP(ILLEGAL_INSN)
    END

    IF COND(REG(Xa), REG(Xb)) THEN
        PC := PC + SX(disp13)
    ELSE
        PC := PC + 4
    END

### CSRRC, CSRRCI, CSRRS, CSRRSI, CSRRW, CSRRWI

|31 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|csr|Xa/imm5|fn3|Xd|11100|11|

|fn3|Syntax|Function (but, see note below)|
|:-:|:----:|:-------|
|001|CSRRW|REG(Xd), CSR(csr) := CSR(csr), REG(Xa)|
|010|CSRRS|REG(Xd), CSR(csr) := CSR(csr), REG(Xa) OR CSR(csr)|
|011|CSRRC|REG(Xd), CSR(csr) := CSR(csr), ~REG(Xa) AND CSR(csr)|
|101|CSRRW|REG(Xd), CSR(csr) := CSR(csr), ZX(imm5)|
|110|CSRRS|REG(Xd), CSR(csr) := CSR(csr), ZX(imm5) OR CSR(csr)|
|111|CSRRC|REG(Xd), CSR(csr) := CSR(csr), ~ZX(imm5) AND CSR(csr)|

**NOTE.** Updates to CSR(csr) are inhibited if Xa/imm5 = 0.

**NOTE.** Access to an CSR which is not supported by Polaris will result in an illegal instruction trap.

### EBREAK

|31 .. 21|20 |19 .. 7|6 .. 2|1 0|
|:------:|:-:|:-----:|:----:|:-:|
|0|1|0|11100|11|

Triggers a breakpoint by trapping to the system software.  The EBREAK handler is determined by the current MTVEC setting.

The intention for this instruction is to provide a set of software-installed breakpoints for the benefit of a debugger or profiler.

    TRAP(EBREAK)

### ECALL

|31 .. 21|20 |19 .. 7|6 .. 2|1 0|
|:------:|:-:|:-----:|:----:|:-:|
|0|0|0|11100|11|

Performs a system call by trapping to the system software.  The ECALL handler is determined by the current MTVEC setting.

The intention for this instruction is to provide a set of system services to less privileged software modules, such as you'd find in operating systems like Unix or Plan 9 from Bell Labs.

    TRAP(ECALL)

### ERET

|31 .. 29|28 |27 .. 7|6 .. 2|1 0|
|:------:|:-:|:-----:|:----:|:-:|
|0|1|0|11100|11|

Returns from handling a trap of some kind, be it interrupt or exception.

    MSTATUS := (MSTATUS AND $FFFFFFFFFFFEF000) OR RSH(MSTATUS AND $FF8, 3) OR $0206
    PC := MEPC

### FENCE, FENCE.I

|31 .. 28|27 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:------:|:-----:|:----:|:-:|
|0|-/0|0|fn3|0|00011|11|

These instructions are implemented on the Polaris processor, and currently perform no action when executed.  Bits 27..20 of the instruction are ignored if, and only if, fn3 = 0.

|fn3|Syntax|Action|
|:-:|:-----|:-----|
|000|FENCE|No action taken.|
|001|FENCE.I|No action taken.|
|01x|-|Illegal instruction trap.|
|1xx|-|Illegal instruction trap.|

### JAL Xd, disp21

|31 |30 .. 21|20 |19 .. 12|11 .. 7|6 .. 2|1 0|
|:-:|:------:|:-:|:------:|:-----:|:----:|:-:|
|disp[20]|disp[10:1]|disp[11]|disp[19:12]|Xd|11011|11|

    REG(Xd) := PC + 4
    PC := PC + SX(disp21)

### JALR Xd, Xa, disp12

|31 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|disp[11:0]| Xa   | 000    | Xd    |11001 |11 |

    REG(Xd) := PC + 4
    PC := REG(Xa) + SX(disp12)

### L

    LB Xd, Xa, disp12
    LH Xd, Xa, disp12
    LW Xd, Xa, disp12
    LD Xd, Xa, disp12
    LBU Xd, Xa, disp12
    LHU Xd, Xa, disp12
    LWU Xd, Xa, disp12
    LDU Xd, Xa, disp12

|31 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|disp12|Xa|sz|Xd|00000|11|

Retrieves a byte, half-word, word, or double-word from memory at the effective address provided.  For multi-byte quantities, the effective address must be naturally aligned.

|sz  |Syntax|Width (bits)|Sign-Extended|Alignment|
|:--:|:----:|:----------:|:-----------:|:--------|
|000 |LB    |8 |Y||
|001 |LH    |16|Y|EA[0] = 0|
|010 |LW    |32|Y|EA[1:0] = 00|
|011 |LD    |64|Y|EA[2:0] = 000|
|100 |LBU   |8 |N||
|101 |LHU   |16|N|EA[0] = 0|
|110 |LWU   |32|N|EA[1:0] = 00|
|111 |LDU   |64|N|EA[2:0] = 000|

    EA := REG(Xa)+disp12
    IF MISALIGNED(EA) THEN
        TRAP(LOAD_ADDR_MISALIGNED)
    ELSE
        REG(Xd) := SX(MEM(sz, REG(Xa) + disp12))
    END

### LUI Xd, imm32

|31 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:-----:|:----:|:-:|
|imm20   |Xd     |01101 | 11|

    REG(Xd) := SX(imm32) AND $FFFFFFFFFFFFF000

### S

    SB Xa, Xb, disp12
    SH Xa, Xb, disp12
    SW Xa, Xb, disp12
    SD Xa, Xb, disp12

|31 .. 25|24 .. 20|19 .. 15|14 .. 12|11 .. 7|6 .. 2|1 0|
|:------:|:------:|:------:|:-----:|:----:|:-:|
|disp[11:5]|Xb|Xa|sz|disp[4:0]|01000|11|

Stores a byte, half-word, word, or double-word to memory at the provided effective address.  Multi-byte quantities must be aligned to their natural word size.

|sz  |Syntax|Width (bits)|Sign-Extended|Alignment|
|:--:|:----:|:----------:|:-----------:|:--------|
|000 |SB    |8 |Y||
|001 |SH    |16|Y|EA[0] = 0|
|010 |SW    |32|Y|EA[1:0] = 00|
|011 |SD    |64|Y|EA[2:0] = 000|

    EA := REG(Xa) + SX(disp12)
    IF sz > 3 THEN
        TRAP(ILLEGAL_INSN)
    ELSIF MISALIGNED(EA)
        TRAP(STORE_ADDR_MISALIGNED)
    ELSE
        ByteAt(REG(Xa) + SX(disp12)) := REG(Xb)
    END

### WFI

|31 .. 20|19 .. 7|6 .. 2|1 0|
|:------:|:-----:|:----:|:-:|
|000100000010|0|11100|11|

Wait for *any* interrupt to occur, enabled or not.

If the corresponding interrupt enable bit is set in MIE, and that interrupt occurs, Polaris will dispatch the interrupt handler as it normally would.  However, MEPC will be loaded with the instruction address *following* WFI, not of WFI itself.

    WAIT UNTIL MIP <> 0

## Opcode Map
This matrix is constructed by plotting instructions on a 16x16 grid. If we concatenate bits 14..12 and 6..2 of an instruction, we get an 8-bit opcode.  For this opcode map to apply, bits 1..0 of the instruction *must* be 11.

Blank instruction slots are guaranteed to raise an illegal instruction trap if executed.

|  |x0 |x1|x2|x3     |x4   |x5   |x6   |x7|x8  |x9  |xA|xB |xC    |xD |xE     |xF|
|:-|:--|:-|:-|:------|:----|:----|:----|:-|:---|:---|:-|:--|:-----|:--|:------|:-|
|0x|LB |  |  |FENCE  |ADDI |AUIPC|ADDIW|SB|    |    |  |   |ADD\*1|LUI|ADDW\*2|  |
|1x|   |  |  |       |     |     |     |  |BEQ |JALR|  |JAL|\*3   |   |       |  |
|2x|LH |  |  |FENCE.I|SLLI |AUIPC|SLLIW|SH|    |    |  |   |SLL   |LUI|SLLW   |  |
|3x|   |  |  |       |     |     |     |  |BNE |    |  |JAL|CSRRW |   |       |  |
|4x|LW |  |  |       |SLTI |AUIPC|     |SW|    |    |  |   |SLT   |LUI|       |  |
|5x|   |  |  |       |     |     |     |  |    |    |  |JAL|CSRRS |   |       |  |
|6x|LD |  |  |       |SLTIU|AUIPC|     |SD|    |    |  |   |SLTU  |LUI|       |  |
|7x|   |  |  |       |     |     |     |  |    |    |  |JAL|CSRRC |   |       |  |
|8x|LBU|  |  |       |XORI |AUIPC|     |  |    |    |  |   |XOR   |LUI|       |  |
|9x|   |  |  |       |     |     |     |  |BLT |    |  |JAL|CSRRWI|   |       |  |
|Ax|LHU|  |  |       |SRxI |AUIPC|SRxIW|  |    |    |  |   |SRx   |LUI|SRxW   |  |
|Bx|   |  |  |       |     |     |     |  |BGE |    |  |JAL|CSRRSI|   |       |  |
|Cx|LWU|  |  |       |ORI  |AUIPC|     |  |    |    |  |   |OR    |LUI|       |  |
|Dx|   |  |  |       |     |     |     |  |BLTU|    |  |JAL|CSRRCI|   |       |  |
|Ex|LDU|  |  |       |ANDI |AUIPC|     |  |    |    |  |   |AND   |LUI|       |  |
|Fx|   |  |  |       |     |     |     |  |BGEU|    |  |JAL|      |   |       |  |

**NOTES**

- *\*1*  This can be either an ADD or SUB, depending on bit 30 of the instruction.
- *\*2*  This can be either ADDW or SUBW, depending on bit 30 of the instruction.
- *\*3* These instructions comprise ECALL, EBREAK, ERET, WFI, etc.

## CPU Specific Registers

CPU Specific Registers allow the software to customize the operation of the specific model and/or brand of processor.

CSRs are read-only, unless specified otherwise.

### UCYCLE

|63 .. 0|
|:-----:|
|cycles|

This register exposes an up-counter to all privilege levels that counts the number of CPU clock cycles that have elapsed.  Note that this counter *is not* a wall-clock counter like MTIME is.

### UTIME

|63 .. 0|
|:-----:|
|time|

This register exposes a read-only image of the MTIME register to all privilege levels.  Unlike UCYCLE, UTIME *is* intended to be a wall-clock counter.

Knowing these two values allows the software to calculate its clocking efficiency.

### UINSTRET

|63 .. 0|
|:-----:|
|retcyc|

This counter exposes a read-only image of an up-counter that counts the number of retired instructions.

### MCPUID

|63 .. 0|
|:-----:|
|$8000000000040100|

This register identifies Polaris as a 64-bit RISC-V integer processor with RV64I and RV64S compatibility.

### MIMPID

|63 .. 32|31 .. 28|27 .. 24|23 .. 16|15 .. 0|
|:------:|:------:|:------:|:------:|:-----:|
|0|major|minor|patch|source|

This register identifies the vendor and specific implementation of the Polaris processor.

The `source` field identifies the specific vendor.  Processors originating with the Kestrel Computer Project, particularly the Redtail architecture processors, are open-source and can be provided by anyone, especially if programmed into an FPGA development board.  Thus, this field should be set to `$8000` for soft-core implementations.

The `major`, `minor`, and `patch` fields provide a [semantic version](http://semver.org) of the soft-core's implementation.  As of this publication, the following values are defined:

|Model|Architecture|Source|Version|
|:---:|:----------:|:----:|:-----:|
|Polaris|Redtail|$8000|0.0.0|

The `major` field permits up to 16 *major versions* of the interface exposed to the programmer.  A major version is defined as a *backward incompatible* change to the programming interface.  Version 0 implies conformance with the user- and machine-mode instruction sets and CSR layouts as specified by the user spec and supervisor spec.

**NOTE.** A major version *is not* required for changing the *hardware* interface to the processor.  Details such as the choice of front-side bus or physical sizes of caches are more often than not transparent to software running on the computer.  Instead, such details typically influence the physical design of the motherboard or compute board on which the processor resides.  Thus, if such information is required for some reason, your software should consult a hardware description manifest as provided by your processor board provider, such as is described by the SDB specification.

The `minor` field permits up to 16 *minor versions* of the interface exposed to the programmer.  A minor version is defined as *backward compatible* enhancements to the programming interface.  Version 0 implies basic RV64I and machine-mode RV64S instruction sets and memory models only.

The `patch` field permits up to 256 *patches* of the interface exposed to the programmer.  A patch is defined as a *backward compatible* fix to erroneous behavior.  Patch 0 implies conformance with the published RISC-V user spec and supervisor spec documents primarily, and with this specification secondarily.

### MHARTID

|63 .. 4|3 .. 0|
|:-----:|:----:|
|0|id|

This register is used to uniquely identify the **har**dware **t**hread to the querying software.  In single-processor computers, this register should read as zero.  Otherwise, in multi-processor computers, the `id` field will uniquely identify one of 16 Polaris cores.

By convention. processor 0 is responsible for bootstrapping the remainder of the system.

### MSTATUS

|63  |62 .. 22|21 .. 17|16  |15 14|13 12|11 10|9   |8  7|6   |5  4|3   |2  1|0   |
|:--:|:------:|:------:|:--:|:---:|:---:|:---:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|SD  |0       |VM      |MPRV|XS   |FS   |PRV3 |IE3 |PRV2|IE2 |PRV1|IE1 |PRV0|IE0 |

A description of the `MSTATUS` fields follows.

|Field|Writeable?|Default|Description|
|:---:|:--------:|:-----:|:----------|
|IE0|Y|0|1 if interrupts are *currently* enabled for the current operating mode; 0 otherwise.  After an interrupt or trap is taken, this field is *always* reset to 0.|
|PRV0|N|3|Always set to 3, indicating the *current* operating mode is machine-mode.|
|IE1, IE2, IE3|Y|0|1 if interrupts *were* enabled in previous operating modes, prior to any interrupts or exceptions taken; 0 otherwise.|
|PRV1, PRV2, PRV3|N|3|The previous operating modes, just before the interrupt or trap.  Only machine-mode is supported, so these values read 3 and cannot be changed.|
|FS|N|0|Floating-point register state, useful for multitasking.  Note that floating point is not supported by Polaris, so this field always reports as 0.|
|XS|N|0|Extension register state.  Reserved for future definition.  Always 0.|
|MPRV|N|0|Supervisor-level memory privilege flag.  Unused in Polaris.|
|VM|N|0|Virtual Memory model selector.  Polaris does not support virtual- or protected-memory as of this publication, so this field is hardwired to zero.|
|SD|N|0|State Dirty flag.  Set if *any* bit in FS *or* XS fields are 1.  Since Polaris does not support hardware extensions or hardware floating point, this field will always report 0.|

When a trap or interrupt occurs, the previous contents of PRV3 and IE3 are lost, as they take on the values of PRV2 and IE2, respectively.  If more than four levels of protection are required, the system software must take it upon itself to save and restore these flags as appropriate.

Some fields of this register are writable.

### MTVEC

|63 .. 2|1 0|
|:-----:|:-:|
|TVBA|0|

`MTVEC` points to the base address containing the processor's trap/interrupt handlers.  On power-on reset, `MTVEC` is set to $FFFFFFFFFFFFFE00.

**NOTE:** Actual handlers appear here, *not* vectors to the handlers.  You can still implement vectoring in software if this capability is desired.

Which exception handler gets to handle a trap or interrupt will depend on whether the processor was running in user-, supervisor-, hypervisor-, or machine-mode at the time.

|Current mode|Interrupt handler address|
|:----------:|:-----------------------:|
|User|MTVEC+$00|
|Supervisor|MTVEC+$40|
|Hypervisor|MTVEC+$80|
|Machine|MTVEC+$C0|

**NOTE:** Since Polaris does not support user-, supervisor-, or hypervisor-modes of operation, the only interrupt handler the processor will ever invoke is the machine-mode trap handler at MTVEC+$C0.

The TVBA field is writable.

### MTDELEG

|63 .. 0|
|:-----:|
|0|

The *trap delegation* register is intended to tell the processor which traps should be handled by handlers directly in hypervisor, supervisor, or optionally in user-mode.  This allows software to respond to traps and interrupts faster, since they don't need to go through the machine-mode dispatch logic first.

Since Polaris does not support hypervisor, supervisor, or user modes of operation, this register will always answer 0, and cannot be written to.

### MIE

|63  |62 .. 60|59  |58 .. 56|55  |54 .. 52|51  |50 .. 8|7   |6 .. 4|3   |2 .. 0 |
|:--:|:------:|:--:|:------:|:--:|:------:|:--:|:-----:|:--:|:----:|:--:|:-----:|
|IRQ0E|0|IRQ1E|0|IRQ2E|0|IRQ3E|0|MTIE|0|MSIE|0|

This register contains the interrupt *enable* bits.  The fields are as follows:

|Field|Writable?|Default|Description|
|:---:|:-------:|:-----:|:----------|
|IRQ0E, IRQ1E, IRQ2E, IRQ3E|Y|0|1 = External interrupt is allowed to fire; otherwise, it will be masked.|
|MTIE|Y|0|1 = The timer compare interrupt is allowed to fire.  Otherwise, it will be masked.|
|MSIE|Y|0|1 = The inter-core software interrupt is allowed to fire.  Otherwise, it will be masked.|

Except for the bits listed above, all other fields are read-only.

### MTIMECMP

|63 .. 32|31 .. 0|
|:------:|:-----:|
|0|mtimecmp|

RISC-V specifies the presence of a wall-clock timer which can interrupt the processor at programmed intervals.  The processor will set the MTIP bit in the MIP register when `mtimecmp` = MTIME[31:0].  Of course, the interrupt will only be taken if the MTIE bit is set in MIE.

This register is writable.

### MTIME

|63 .. 0|
|:-----:|
|mtime|

This is the current time as understood by the processor.  Note that this is *not* a cycle-counter; it is intended to represent wall-clock time, albeit with higher precision.

When the processor resets, MTIME takes on the value of 0.  If used also for real-time clock purposes, it may be written with a new value.

The rate this counter increments at is determined by the circuit using the Polaris processor core.  The Kestrel-3 increments this counter at 10MHz.

This register is writable.

### MSCRATCH

|63 .. 0|
|:-----:|
|scratch|

This register is unused by Polaris.  It is intended to serve as a storage location for interrupt- and trap-handlers when saving and restoring state.

For example, let's consider a system in which you're running software in user-mode, and it traps to the kernel for a system call.  To save the complete set of CPU registers when changing tasks or processes, for example, you cannot just use one of the registers as a supervisor-level stack pointer without destroying some state that the user-mode process potentially depends on.  Similarly, it's ill-advised to re-use the user-level stack pointer, since access to the stack is (for all you know) what generated the trap in the first place.

To make progress, a typical exception handler will save the current stack pointer in MSCRATCH, reload the stack pointer with the supervisor, hypervisor, or machine-mode stack pointer (presumably placed in a well-known location and free from MMU interference), and only then save all the user-mode registers there-in.

Likewise, when restoring context, MSCRATCH plays a role as a cache for the user-mode stack pointer.  This is because the user-mode stack pointer is the *last* thing you reload prior to invoking ERET.  Otherwise, you run the risk of corrupting user-mode state.

This register is writable.

### MEPC

|63 .. 2|1 0|
|:-----:|:--|
|epc|0|

When a trap or interrupt occurs, MEPC will be loaded with the address that the processor must return to to make progress after the trap or interrupt has been handled.

This register is writable.

### MCAUSE

|63  |62 .. 4|3 .. 0|
|:--:|:-----:|:----:|
|IP  |0|cause|

The fields are as follows:

|Field|Writable?|Default|Description|
|:---:|:-------:|:-----:|:----------|
|IP|Y|0|1 = the exception handler has been invoked as a result of an external interrupt.  0 = the exception is handling a trap.|
|cause|Y|0|One of 16 possible interrupt or trap causes. See table in Traps section, below.|

Except for the fields listed above, no other field is writable.

### MBADADDR

|63 .. 0|
|:-----:
|addr|

When a trap is taken, this register will contain any fault-related effective address that the processor was trying to use at the time.

This register is writable.

### MIP

|63  |62 .. 60|59  |58 .. 56|55  |54 .. 52|51  |50 .. 8|7   |6 .. 4|3   |2 .. 0 |
|:--:|:------:|:--:|:------:|:--:|:------:|:--:|:-----:|:--:|:----:|:--:|:-----:|
|IRQ0P|0|IRQ1P|0|IRQ2P|0|IRQ3P|0|MTIP|0|MSIP|0|

This register contains the interrupt *pending* bits.  The fields are as follows:

|Field|Writable?|Default|Description|
|:---:|:-------:|:-----:|:----------|
|IRQ0P, IRQ1P, IRQ2P, IRQ3P|Y|0|1 = External interrupt has fired since this field was last reset to zero.|
|MTIP|Y|0|1 = At least one timer compare interrupt fired since this field was last reset to zero.|
|MSIP|Y|0|1 = At least one software interrupt, generated from another processor core, has fired since this field was last reset to zero.  For single-processor systems, you'll only see this happen if the core sets its own interrupt pending bit.|

Except for the fields listed above, no other field is writable.

### MTOHOST

| 63 .. 0 |
|:-------:|
|output|

A general purpose, 64-bit output port.  This can be used for general peripheral control, or as its name implies, to establish a 4-way, asynchronous, message-passing handshake with a host computer.  This host computer would typically service requests from the Polaris processor.  Polaris does not currently interpret the meaning of this register's contents.

This register is writable.

### MFROMHOST

| 63 .. 0 |
|:-------:|
|input|

A general purpose, 64-bit input port.  This can be used for general peripheral sensing, or as its name implies, to establish a 4-way, asynchronous, message-passing handshake with a host computer.  This Polaris-based computer would typically handle requests and message replies from the host computer to which it's attached.  Polaris does not currently interpret the meaning of this register's contents.

This register is read-only.

## Interrupt and Trap Reference

When Polaris takes a trap or interrupt, the following sequence of actions takes place:

    ModeSwitch(cause, isInterrupt):
        MEPC := PC
        MCAUSE := cause OR LSH(isInterrupt, 63)
        PC := MTVEC + LSH(MSTATUS AND $06, 5)
        MSTATUS := (MSTATUS AND $FFFFFFFFFFFEF000) OR LSH(MSTATUS AND $1FF, 3) OR 6

The high-level bit in `MCAUSE` determines which type of event happened, allowing an interrupt or trap handler to quickly determine a course of action by a simple `BLT` or `BGE` instruction.

    TRAP(cause):
        ModeSwitch(cause, 0)

    INT(cause):
        ModeSwitch(cause, 1)

### Traps

The following causes for traps are supported:

|Cause Code|Description|Why|
|:--------:|:----------|:--|
|0|Instruction address misaligned.|PC[1:0] are not zero.|
|1|Instruction access fault.|Not currently generated.|
|2|Illegal instruction.|Fetched instruction does not conform to the bit patterns of the currently supported instruction set.  See next section for details.|
|3|EBREAK or hardware breakpoint.|The EBREAK instruction was executed.|
|4|Load address misaligned.|Not generated for byte-wide accesses.  For half-word accesses, EA[0] must be clear.  For word accesses, EA[1:0] must be clear.  For double-word accesses, EA[2:0] must be clear.|
|5|Load access fault.|Not currently generated.|
|6|Store address misaligned.|Not generated for byte-wide accesses.  For half-word accesses, EA[0] must be clear.  For word accesses, EA[1:0] must be clear.  For double-word accesses, EA[2:0] must be clear.|
|7|Store access fault.|Not currently generated.|
|8|ECALL from U-mode.|Not currently generated.|
|9|ECALL from S-mode.|Not currently generated.|
|10|ECALL from H-mode.|Not currently generated.|
|11|ECALL from M-mode.|ECALL instruction invoked.|
|12|Reserved.|Not currently generated.|
|13|Reserved.|Not currently generated.|
|14|Reserved.|Not currently generated.|
|15|Reserved.|Not currently generated.|

### Interrupts

The following causes for interrupts are supported, with IRQ0 having the highest priority.

|Cause Code|Description|Why|
|:--------:|:----------|:--|
| 0|Reserved.|Not currently generated.|
| 1|Reserved.|Not currently generated.|
| 2|Reserved.|Not currently generated.|
| 3|Reserved.|Not currently generated.|
| 4|Reserved.|Not currently generated.|
| 5|Reserved.|Not currently generated.|
| 6|Reserved.|Not currently generated.|
| 7|Reserved.|Not currently generated.|
| 8|Reserved.|Not currently generated.|
| 9|Reserved.|Not currently generated.|
|10|Reserved.|Not currently generated.|
|11|Reserved.|Not currently generated.|
|12|IRQ3|External IRQ 3 triggered.|
|13|IRQ2|External IRQ 2 triggered.|
|14|IRQ1|External IRQ 1 triggered.|
|15|IRQ0|External IRQ 0 triggered.|
