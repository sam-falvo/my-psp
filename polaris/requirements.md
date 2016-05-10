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
- Split instruction and data buses enables use in a Harvard or Von Neumann architecture design.
- 64-bit data and address widths
- 51 instructions
- 31 integer registers (X0 hardwired to zero)
- Interrupt capability
- Illegal instruction traps
- Multiple bus master support
- Pipelined architecture
- 64-bit general purpose output port
- 64-bit general purpose input port

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

    Integer Registers       Core Specific Registers

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

### ADD Xd, Xa, Xb

    REG(Xd) := REG(Xa) + REG(Xb)

### ADDI Xd, Xa, imm12

    REG(Xd) := REG(Xa) + SX(imm12)

### AND Xd, Xa, Xb

    REG(Xd) := REG(Xa) + REG(Xb)

### ANDI Xd, Xa, imm12

    REG(Xd) := REG(Xa) + SX(imm12)

### AUIPC Xd, imm32

    REG(Xd) := PC + (SX(imm32) AND $FFFFFFFFFFFFF000)

### BEQ Xa, Xb, disp13

    IF REG(Xa) = REG(Xb) THEN
        PC := PC + 2*disp13
    ELSE
        PC := PC + 4
    END

### BGE Xa, Xb, disp13

    IF REG(Xa) >= REG(Xb) THEN
        PC := PC + 2*disp13
    ELSE
        PC := PC + 4
    END

### BGEU Xa, Xb, disp13

    IF REG(Xa) >= REG(Xb) THEN
        PC := PC + 2*disp13
    ELSE
        PC := PC + 4
    END

### BLT Xa, Xb, disp13

    IF REG(Xa) < REG(Xb) THEN
        PC := PC + 2*disp13
    ELSE
        PC := PC + 4
    END

### BLTU Xa, Xb, disp13

    IF REG(Xa) < REG(Xb) THEN
        PC := PC + 2*disp13
    ELSE
        PC := PC + 4
    END

### BNE Xa, Xb, disp13

    IF REG(Xa) <> REG(Xb) THEN
        PC := PC + 2*disp13
    ELSE
        PC := PC + 4
    END

### CSRRC Xa, csr

    IF Xa > 0 THEN
        tmp := CSR(csr)
        CSR(csr) := CSR(csr) AND ~REG(Xa)
        REG(Xa) := tmp
    ELSE
        // no operation
    END

### CSRRCI imm5, csr

    IF imm5 > 0 THEN
        tmp := CSR(csr)
        CSR(csr) := CSR(csr) AND ~ZX(imm5)
        REG(Xa) := tmp
    ELSE
        // no operation
    END

### CSRRS Xa, csr

    IF Xa > 0 THEN
        tmp := CSR(csr)
        CSR(csr) := CSR(csr) OR REG(Xa)
        REG(Xa) := tmp
    ELSE
        // no operation
    END

### CSRRSI imm5, csr

    IF imm5 > 0 THEN
        tmp := CSR(csr)
        CSR(csr) := CSR(csr) OR ZX(imm5)
        REG(Xa) := tmp
    ELSE
        // no operation
    END

### CSRRW Xa, csr

    IF Xa > 0 THEN
        tmp := CSR(csr)
        CSR(csr) := REG(Xa)
        REG(Xa) := tmp
    ELSE
        // no operation
    END

### CSRRWI imm5, csr

    IF imm5 > 0 THEN
        tmp := CSR(csr)
        CSR(csr) := imm5
        REG(Xa) := tmp
    ELSE
        // no operation
    END

### EBREAK

    offset := LSH(MSTATUS AND $06, 5)
    MEPC := PC
    MCAUSE := (ebreak)
    MSTATUS := (MSTATUS AND $FFFFFFFFFFFEF000) OR LSH(MSTATUS AND $1FF, 3) OR 6
    PC := MTVEC + offset

### ECALL

    offset := LSH(MSTATUS AND $06, 5)
    MEPC := PC
    MCAUSE := (ecall)
    MSTATUS := (MSTATUS AND $FFFFFFFFFFFEF000) OR LSH(MSTATUS AND $1FF, 3) OR 6
    PC := MTVEC + offset

### ERET

    MSTATUS := (MSTATUS AND $FFFFFFFFFFFEF000) OR RSH(MSTATUS AND $FF8, 3) OR $0206
    PC := MEPC

### JAL Xd, disp21

    REG(Xd) := PC + 4
    PC := PC + 2 * SX(disp21)

### JALR Xd, Xa, disp12

    REG(Xd) := PC + 4
    PC := REG(Xa) + SX(disp12)

### LB Xd, Xa, disp12

    REG(Xd) := SX(ByteAt(REG(Xa) + disp12))

### LBU Xd, Xa, disp12

    REG(Xd) := ZX(ByteAt(REG(Xa) + disp12))

### LD Xd, Xa, disp12

    REG(Xd) := SX(DWordAt(REG(Xa) + disp12))

### LDU Xd, Xa, disp12

    REG(Xd) := ZX(DWordAt(REG(Xa) + disp12))

### LH Xd, Xa, disp12

    REG(Xd) := SX(HWordAt(REG(Xa) + disp12))

### LHU Xd, Xa, disp12

    REG(Xd) := ZX(HWordAt(REG(Xa) + disp12))

### LUI Xd, imm32

    REG(Xd) := SX(imm32) AND $FFFFFFFFFFFFF000

### LW Xd, Xa, disp12

    REG(Xd) := SX(WordAt(REG(Xa) + disp12))

### LWU Xd, Xa, disp12

    REG(Xd) := ZX(WordAt(REG(Xa) + disp12))

### OR Xd, Xa, Xb

    REG(Xd) := REG(Xa) OR REG(Xb)

### ORI Xd, Xa, imm12

    REG(Xd) := REG(Xa) OR SX(imm12)

### SB Xa, Xb, disp12

    ByteAt(REG(Xb) + SX(disp12)) := REG(Xa)

### SD Xa, Xb, disp12

    DWordAt(REG(Xb) + SX(disp12)) := REG(Xa)

### SH Xa, Xb, disp12

    HWordAt(REG(Xb) + SX(disp12)) := REG(Xa)

### SLL Xd, Xa, Xb

    REG(Xd) := LSH(REG(Xa), REG(Xb) AND 63)

### SLLI Xd, Xa, imm12

    IF imm12 < 0 OR imm12 >= 64 THEN
        TRAP(ILLEGAL_INSN)
    ELSE
        REG(Xd) := LSH(REG(Xa), imm12)
    END

### SLT Xd, Xa, Xb

    IF REG(Xa) < REG(Xb) THEN
        REG(Xd) := 1
    ELSE
        REG(Xd) := 0
    END

### SLTI Xd, Xa, imm12

    IF REG(Xa) < SX(imm12) THEN
        REG(Xd) := 1
    ELSE
        REG(Xd) := 0
    END

### SLTIU Xd, Xa, imm12

    IF REG(Xa) < SX(imm12) THEN
        REG(Xd) := 1
    ELSE
        REG(Xd) := 0
    END

### SLTU Xd, Xa, Xb

    IF REG(Xa) < REG(Xb) THEN
        REG(Xd) := 1
    ELSE
        REG(Xd) := 0
    END

### SRA Xd, Xa, Xb

    REG(Xd) := ASR(REG(Xa), REG(Xb) AND 63)

### SRAI Xd, Xa, imm12

    IF imm12 < 0 OR imm12 >= 64 THEN
        TRAP(ILLEGAL_INSN)
    ELSE
        REG(Xd) := ASR(REG(Xa), imm12)
    END

### SRL Xd, Xa, Xb

    REG(Xd) := RSH(REG(Xa), REG(Xb) AND 63)

### SRLI Xd, Xa, imm12

    IF imm12 < 0 OR imm12 >= 64 THEN
        TRAP(ILLEGAL_INSN)
    ELSE
        REG(Xd) := RSH(REG(Xa), imm12)
    END

### SUB Xd, Xa, Xb

    REG(Xd) := REG(Xa) - REG(Xb)

### SW Xa, Xb, disp12

    WordAt(REG(Xb) + SX(disp12)) := REG(Xa)

### WFI

    WAIT UNTIL (MIP AND MIE) <> 0

### XOR Xd, Xa, Xb

    REG(Xd) := REG(Xa) XOR REG(Xb)

### XORI Xd, Xa, imm12

    REG(Xd) := REG(Xa) XOR SX(imm12)
