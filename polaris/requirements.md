# Polaris Requirements

Polaris is the first Verilog implementation of a RISC-V RV64I and RV64S instruction set compatible processor.  It's intended to serve as the core processor for first-generation Kestrel-3 implementations.  As of this writing, first-generation implementations include:

- Digilent Nexys-2 FPGA development board.

The Polaris specification includes by reference several external specifications that determines RISC-V instruction set compatibility and/or hardware interoperability in the open-hardware community:
 
- Where this document references the *user specification* or *user spec*, please refer to [The RISC-V Instruction Set Manual, Volume I: User-Level ISA, Version 2.0](http://riscv.wpengine.com/specifications/).

- Where this document references the *supervisor specification* or *supervisor spec*, please refer to [The Draft RISC-V Instruction Set Manual, Volume II: Privileged Architecture, Version 1.7](http://riscv.wpengine.com/specifications/privileged-isa/).

- Where this document references the *Wishbone specification* or *Wishbone spec*, please refer to the [Wishbone B4: WISHBONE System-on-Chip (SOC) Interconnection Architecture for Portable IP Cores](http://cdn.opencores.org/downloads/wbspec_b4.pdf) specification.

Requirements will be documented with the following notation:

- (Rx) This requirement MUST be held.
- (Rx.1) This requirement MAY be a sub-requirement of Rx, but doesn't have to be.
- (Rx.2) Requirements invariably change over time.  We use this notation because renumbering all the requirements in the document will become burdensome beyond a certain scope.
- (Rx.2.1) It's also provides an effectively infinite namespace for related requirements.

After requirements have been settled on and other development phases have been started, requirements may need updating, particularly as *new* requirements are discovered later in the project.  Such requirements will be identified using D instead of R, standing for *derived* requirements.

## High-Level 1st Generation Kestrel-3 Architecture

    +---------+
    |         |
    |         |=====\=========\========\=========\===========\=============\
    |         |     ||        ||       ||        ||          ||            ||
    |         |  +------+  +-----+  +------+  +------+  +----------+  +---------+
    | Polaris |  | GPIA |  | KIA |  | MGIA |  | SRAM |  | Boot ROM |  | SDB ROM |
    |         |  +------+  +-----+  +------+  +------+  +----------+  +---------+
    |         |     ||        ||       ||        ||          ||            ||
    |         |=====/=========/========/=========/===========/=============/
    |         |
    +---------+

The Polaris processor serves as the Kestrel's prime mover.  Nothing in the Kestrel-3 happens without processor involvement at some level.  In this respect, the Kestrel-3 is architecturally identical to the Kestrel-2, with the key value additions being a 64-bit CPU and access to memory external to the hosting FPGA.

### General Purpose Interface Adapter (GPIA)

The Kestrel-2 utilized a 16-bit GPIA device, with an input port at byte offset 0, and an output port at byte offset 2.  The Kestrel-3 uses a 64-bit version of the GPIA, known as the GPIA-2.  The input port is at byte offset 0, and the output port at byte offset 8.  These devices are otherwise architecturally the same.

Currently, the GPIA(-2) offers the CPU control over mass storage through an SD card interface configured as an SPI slave peripheral.

### Keyboard Interface Adapter (KIA)

The Kestrel-2 and Kestrel-3 Keyboard Interface Adapter (KIA) are identical in function and memory layout.  Please [refer to its datasheet](https://github.com/KestrelComputer/kestrel/tree/master/cores/KIA/doc) for details on its programming interface.

For pragmatic reasons, the KIA used in the `e` emulator is a *10-bit* device, supporting the libSDL2-compatible representation of low-level keyboard scancodes.  This implementation of the KIA is known as the KIA-2.  Bits 8 and 9 of the scancode can be read as the top two bits of the status register.  The KIA-2 is not intended for actual hardware implementation.

### Monochrome Graphics Interface Adapter (MGIA)

The Kestrel-2 utilized a version of the MGIA which offered a 640x200 monochrome, bitmapped display.  The Kestrel-3 uses an enhancement called the MGIA-2, which expands the resolution to 640x480, but is otherwise identical.  This core offers no addressible registers, and locates its video space (for the Nexys-2 implementation) at the fixed address range $FF0000-$FF95FF.

### External RAM

The largest capacity memory accessible to Polaris, by far, is the external RAM.  On the Nexys-2 board, 16MB of "cellular RAM" is provided with a bus protocol compatible with JEDEC-standard static RAMs.  Some cheaper FPGA boards provide 8MB of SDRAM.  Others commonly available provide 64MB of SDRAM.

- (R1.1) Polaris MUST be able to address external static RAM up to 16MB minimum.

### Boot ROM

The most commonly available FPGA development boards all seem to offer at least 1MB of flash ROM for use by the FPGA.  At this time, the compiler tools for the Kestrel-3 assume 1MB of ROM is available for programming at time of synthesis.

- (R1.2)  Polaris MUST be able to address external ROM up to 1MB minimum.

Through some experimentation and natural evolution with the `e` emulator, a memory map like the following evolved.

|From            | To            |What's There|
|:--------------:|:-------------:|:----------:|
|0000000000000000|0000000000FFFFFF|RAM|
|1000000000000000|100000000000000F|GPIA-2|
|1000000000000010|1FFFFFFFFFFFFFFF|GPIA-2 (mirror)|
|2000000000000000|2000000000000001|KIA|
|2000000000000002|2FFFFFFFFFFFFFFF|KIA (mirror)|
|3000000000000000|30000000000001FF|SDB ROM|
|3000000000000200|3FFFFFFFFFFFFFFF|SDB ROM (mirror)|
|4000000000000000|EFFFFFFFFFFFFFFF|unspecified|
|FFFFFFFFFFF00000|FFFFFFFFFFFFFFFF|Boot ROM|

Because ROM appears in high memory, interrupt and other RISC-V machine-mode resources must appear there as well.

- (R1.3) Per the supervisor spec, Polaris MUST execute its first instruction at $FFFFFFFFFFFFFF00.

- (R1.4) Per the supervisor spec, the `mtvec` register MUST reset to $FFFFFFFFFFFFFE00 upon hard reset.

- (R1.5) Polaris MUST allow a machine-mode program to change the `mtvec` contents.  All writable bits, as specified by the supervisor spec, must be supported.

### SDB ROM

The SDB ROM is used to provide descriptors intended to be compatible with the Self-Describing Bus protocol.  These descriptors help both the system firmware and any loaded operating system to auto-detect what the hardware-level configuration of the Kestrel-3 actually is.

For example, as of this writing, version 1.01 of Kestrel Forth ROM simply *assumes* the MGIA video framebuffer at $FF0000.  However, it could be enhanced later to look it up in the SDB ROM, so that the same firmware image could run on any FPGA development board.  Similarly, it could use the same SDB ROM image to determine if enhanced capabilities exist, such as color, sound, etc.

The format of the SDB image is beyond the scope of this document.

