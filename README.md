# Complete CPU

## Summary
The purpose of this project is to implement and simulate a complete Semi-RISC CPU that consists of a data path, control unit, and a reset circuit in correspondance with the [CPU specification document](https://www.ee.torontomu.ca/~courses/coe608/labs/CPU_Specification.pdf).

## Features
### 1. Data Path
- Interconnects the PC, IR, Registers (A, B, C, Z), ALU, data memory unit, and required multiplexers
- Exposes PC, IR, and Register (A, B, C, Z) outputs as top-level outputs to support testing and waveform observation
- Uses separate clocks: 
    - `Clk` for sequential elements (PC/IR/registers)
    - `mClk` for data memory timing
- Supports load/store flow with IR-latched addressing through a reducer (RED) (IR[7:0]) feeding the data memory address
- Provides data flow control via `DATA_MUX`, `REG_MUX`, `A_MUX`, `B_MUX`, `IM_MUX1`, and `IM_MUX2` to route operands and results
- Includes upper and lower zero extenders (UZE and LZE) for immediates and load upper immediate (LUI) behaviour (lower/upper zero extension for IR[15:0])
- Routes ALU results and memory data onto an internal data bus used for register writes
- Designed to be driven by the control unit’s T0/T1/T2 sequencing for multi-cycle instruction execution

### 2. Control Unit
- Drives all datapath control signals for registers, ALU, and multiplexers
- Uses a three-state sequencer (`T0`, `T1`, `T2`) and exposes the `T` pulse outputs for instruction timing
- Decodes instruction fields (`INST[31:28]` opcode and `INST[27:24]` function) plus status flags (`C`, `Z`) to select operations
- Generates `wen` and `en` for data memory with setup/hold timing aligned to `Clk` and `mClk` during load/store sequences
- Honors `enable` gating: when deasserted, the control unit returns to `T0` and holds until re-enabled
- Coordinates with the reset circuit for `Clr_PC` generation outside the control unit

### 3. Reset Circuit

