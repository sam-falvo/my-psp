# Polaris Processor Core

## Description

Polaris is the first member of a family of processors conforming to the Redtail Architecture.  It aims to solve a broad range of small systems and peripheral control problems at minimum cost to the user.  It implements the RISC-V RV64I and RV64S instruction sets.

The Polaris specification includes by reference several external specifications that determines RISC-V instruction set compatibility and/or hardware interoperability in the open-hardware community:
 
- Where this document references the *user specification* or *user spec*, please refer to [The RISC-V Instruction Set Manual, Volume I: User-Level ISA, Version 2.0](http://riscv.wpengine.com/specifications/).

- Where this document references the *supervisor specification* or *supervisor spec*, please refer to [The Draft RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Version 1.7](http://riscv.wpengine.com/specifications/privileged-isa/).

- Where this document references the *Wishbone specification* or *Wishbone spec*, please refer to the [Wishbone B4: WISHBONE System-on-Chip (SOC) Interconnection Architecture for Portable IP Cores](http://cdn.opencores.org/downloads/wbspec_b4.pdf) specification.

- Where this document references the *SDB specification* or *SDB spec*, please refer to the [Self-Describing Bus V1.1](http://www.ohwr.org/projects/sdb/wiki) specification.

## Features

- Wishbone B3 and B4 compatible front-side bus.
- 64-bit data and address widths
- Eight select lines enables memory to be addressed at byte granularity.
- 51 instructions
- 31 integer registers (X0 hardwired to zero)
- 19 CPU Specific Registers (CSRs)
- External interrupt capability
- Most instructions execute in a single clock cycle.
- Multiple bus master support
- Pipelined architecture
- 64-bit general purpose output port
- 64-bit general purpose input port
- Machine-Mode Only.

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

### CSRRC Xd, Xa, csr

    REG(Xd) := CSR(csr)
    IF Xa > 0 THEN
        CSR(csr) := CSR(csr) AND ~REG(Xa)
    END

### CSRRCI Xd, imm5, csr

    REG(Xd) := CSR(csr)
    IF imm5 > 0 THEN
        CSR(csr) := CSR(csr) AND ~ZX(imm5)
    END

### CSRRS Xd, Xa, csr

    REG(Xd) := CSR(csr)
    IF Xa > 0 THEN
        CSR(csr) := CSR(csr) OR REG(Xa)
    END

### CSRRSI Xd, imm5, csr

    REG(Xd) := CSR(csr)
    IF imm5 > 0 THEN
        CSR(csr) := CSR(csr) OR ZX(imm5)
    END

### CSRRW Xd, Xa, csr

    REG(Xd) := CSR(csr)
    IF Xa > 0 THEN
        CSR(csr) := REG(Xa)
    END

### CSRRWI Xd, imm5, csr

    REG(Xd) := CSR(csr)
    IF imm5 > 0 THEN
        CSR(csr) := ZX(imm5)
    END

### EBREAK

    TRAP(EBREAK)

### ECALL

    TRAP(ECALL)

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

## CPU Specific Registers

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

This register is read-only.

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

This register is read-only.

### MHARTID

|63 .. 4|3 .. 0|
|:-----:|:----:|
|0|id|

This register is used to uniquely identify the **har**dware **t**hread to the querying software.  In single-processor computers, this register should read as zero.  Otherwise, in multi-processor computers, the `id` field will uniquely identify one of 16 Polaris cores.

By convention. processor 0 is responsible for bootstrapping the remainder of the system.

This register is read-only.

### MSTATUS

|63  |62 .. 22|21 .. 17|16  |15 14|13 12|11 10|9   |8  7|6   |5  4|3   |2  1|0   |
|:--:|:------:|:------:|:--:|:---:|:---:|:---:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|SD  |0       |VM      |MPRV|XS   |FS   |PRV3 |IE3  |PRV2|IE2 |PRV1|IE1 |PRV0|IE0|

A description of the `MSTATUS` fields follows.

|Field|Writeable?|Default|Description|
|:---:|:--------:|:-----:|:----------|
|IE0|Y|0|1 if interrupts are *currently* enabled for the current operating mode; 0 otherwise.  After an interrupt or trap is taken, this field is *always* reset to 0.|
|PRV0|Y|3|0, 1, 2, or 3 if the *current* operating mode is user, supervisor, hypervisor, or machine mode.  After an interrupt or trap is taken, this field is *always* reset to 3.  Note that the `MTDELEG` register is ignored by Polaris, as Polaris does not support hypervisor, supervisor, or user modes at this time.|
|IE1|Y|0|1 if interrupts *were* enabled in the previous operating mode, prior to the trap or interrupt; 0 otherwise.|
|PRV1|Y|3|The previous operating mode, just before the interrupt or trap.|
|IE2, IE3|Y|0|Further stacked versions of the interrupt enable flag.  **NOTE:** The ERET instruction will *always* set IE3.|
|PRV2, PRV3|Y|3|Further stacked versions of the CPU operating mode.  **NOTE:** The ERET instruction will *always* set PRV3 to 0 (user-mode).|
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

The TVBA field is writable.

### MTDELEG

|63 .. 0|
|:-----:|
|0|

The *trap delegation* register is intended to tell the processor which traps should be handled by handlers directly in hypervisor, supervisor, or optionally in user-mode.  This allows software to respond to traps and interrupts faster, since they don't need to go through the machine-mode dispatch logic first.

Since Polaris does not support hypervisor, supervisor, or user modes of operation, this register will always answer 0, and cannot be written to.

This register is read-only.

### MIE

|63 .. 8|7   |6   |5   |4   |3   |2   |1   |0   |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|0|MTIE|0|0|0|MSIE|0|0|0|

This register contains the interrupt *enable* bits.  The fields are as follows:

|Field|Writable?|Default|Description|
|:---:|:-------:|:-----:|:----------|
|MTIP|Y|0|1 = The timer compare interrupt is allowed to fire.  Otherwise, it will be masked.|
|MSIP|Y|0|1 = The inter-core software interrupt is allowed to fire.  Otherwise, it will be masked.|

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

This register is writable.

### MSCRATCH

|63 .. 0|
|:-----:|
|scratch|

This register is unused by Polaris; however, it is intended to serve as a storage location for interrupt- and trap-handlers when saving state.

For example, let's consider a system in which you're running software in user-mode, and it traps to the kernel for a system call.  To save the complete set of CPU registers when changing tasks or processes, for example, you cannot just use one of the registers as a supervisor-level stack pointer without destroying some state that the user-mode process potentially depends on.  Similarly, it's ill-advised to re-use the user-level stack pointer, since access to the stack is (for all you know) what generated the trap in the first place.

To make progress, a typical exception handler will save the stack pointer in MSCRATCH, reload the stack pointer with the supervisor, hypervisor, or machine-mode stack pointer (presumably placed in a well-known location and free from MMU interference), and only then save all the user-mode registers there-in.

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

### MBADADDR

|63 .. 0|
|:-----:
|addr|

When a trap is taken, this register will contain any fault-related effective address that the processor was trying to use at the time.

This register is writable.

### MIP

|63 .. 8|7   |6   |5   |4   |3   |2   |1   |0   |
|:-----:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|0|MTIP|0|0|0|MSIP|0|0|0|

This register contains the interrupt *pending* bits.  The fields are as follows:

|Field|Writable?|Default|Description|
|:---:|:-------:|:-----:|:----------|
|MTIP|Y|0|1 = At least one timer compare interrupt fired since this field was last reset to zero.|
|MSIP|Y|0|1 = At least one software interrupt, generated from another processor core, has fired since this field was last reset to zero.  For single-processor systems, you'll only see this happen if the core sets its own interrupt pending bit.|

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

The high-level bit in `MSTATUS` determines which type of event happened, allowing an interrupt or trap handler to quickly determine a course of action by a simple `BLT` or `BGE` instruction.

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
tbd
