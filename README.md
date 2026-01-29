# Complete CPU
## Summary
The purpose of this project is to use Altera Quartus II to implement and simulate a complete semi-RISC CPU that consists of a data path, control unit, and a reset circuit in correspondance with the [CPU specification document](https://www.ee.torontomu.ca/~courses/coe608/labs/CPU_Specification.pdf).

## Features
### 1. Data Path
- Interconnects the PC, IR, registers (A, B, C, Z), ALU, data memory unit, and required multiplexers
- Exposes PC, IR, and register (A, B, C, Z) outputs as top-level outputs to support testing and waveform observation
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
- Forces `enable_PD` low and `clr_PC` high when `reset` is asserted, driving the control unit to `T0` and clearing the PC
- Uses a 4-cycle counter (`clk0`, `clk1`, `clk2`, `clk3`) to hold reset sequencing before releasing the CPU
- Deasserts `clr_PC` and re-enables the datapath/control after the reset delay window
- Implemented synchronously on the rising edge of `Clk`

## Simulation
### Description
- Uses a top-level simulation entity that instantiates the CPU and instruction memory for waveform-based testing
- The entity exposes `T` state, PC, IR, and register outputs to simplify debugging during simulation
- Instruction memory is provided via the MegaCore RAM block (`system_memory`) initialized with a `.mif` file
- The CPU instance (`cpu1`) connects the datapath, the control unit, and the reset circuit as the complete system under test
- Two clocks are used in simulation: 
    - `memClk` for instruction/data memory
    - `cpuClk` for the CPU core (CPU clock runs at least 2× slower than memory clock)
- PC uses the lower address bits to index 64×32 instruction memory words (6-bit address)

### Results
Refer to this document: [CPU System Memory Implementation and Functional Simulations](https://github.com/cthanges/Complete-CPU/blob/main/CPU%20System%20Memory%20Implementation%20and%20Functional%20Simulations.pdf).