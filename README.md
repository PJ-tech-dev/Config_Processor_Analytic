# Configurable Processor Performance Monitoring and Analytics IP

<p align="center">
  <strong>A Parameterized Hardware IP for Processor Performance Monitoring, Microarchitectural Event Analysis, and Runtime Analytics</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Language-Verilog-blue" />
  <img src="https://img.shields.io/badge/Design-RTL%20IP-green" />
  <img src="https://img.shields.io/badge/Target-FPGA%20%7C%20ASIC-orange" />
  <img src="https://img.shields.io/badge/Domain-VLSI%20%7C%20Computer%20Architecture-purple" />
</p>

---

## 1. Project Overview

The **Configurable Processor Performance Monitoring and Analytics IP** is a hardware-based monitoring subsystem developed to evaluate processor execution behavior through real-time event collection, performance counter accumulation, and hardware-based metric computation.

The design provides visibility into processor execution by monitoring architectural and microarchitectural events directly from internal processor signals.

Unlike software-only profiling approaches, this IP uses dedicated RTL hardware to capture processor events and calculate performance indicators with minimal dependence on external software instrumentation.

The architecture is organized into independent functional modules for event collection, profiling-mode control, event counting, metric computation, and register-based data access.

The project includes two implementation paths:

- **Standalone Analytics IP:** A modular, parameterized monitoring architecture with dedicated verification testbenches.
- **Processor-Integrated Analytics:** Integration of the analytics subsystem with a pipelined RISC-V processor through internal processor probe signals.

## 2. Project Objectives

- Develop a configurable hardware performance monitoring IP using Verilog HDL.
- Capture architectural and microarchitectural processor events.
- Implement dedicated hardware performance counters.
- Support architectural and microarchitectural profiling modes.
- Monitor pipeline activity, hazards, stalls, bubbles, flushes, and forwarding events.
- Calculate IPC, CPI, pipeline efficiency, stage utilization, and other performance metrics.
- Implement privilege-mode and execution-context filtering interfaces.
- Provide register-based access to collected performance information.
- Develop a reusable architecture suitable for FPGA and ASIC-oriented processor integration.

## 3. System Architecture

The analytics IP follows a modular hardware architecture.

```text
              PROCESSOR / RISC-V CORE
                       |
                       |
                Processor Probes
                       |
                       v
              +--------------------+
              |   EVENT COLLECTOR  |
              |--------------------|
              | Instruction Events |
              | Hazard Detection   |
              | Pipeline Activity  |
              | Privilege Filtering|
              | Context Filtering |
              +--------------------+
                       |
                       v
              +--------------------+
              |  MODE CONTROLLER   |
              |--------------------|
              | Architectural Mode |
              | Microarchitectural |
              | Profiling Control  |
              +--------------------+
                       |
                       v
              +--------------------+
              |   EVENT COUNTER    |
              |--------------------|
              | Cycle Counter      |
              | Instruction Counter|
              | Branch / Memory    |
              | Hazard Counters    |
              | Pipeline Counters  |
              +--------------------+
                       |
                       v
              +--------------------+
              |  ANALYTICS ENGINE  |
              |--------------------|
              | IPC / CPI          |
              | Pipeline Efficiency|
              | Stage Utilization  |
              | Forwarding Metrics |
              | Hazard Distribution|
              +--------------------+
                       |
                       v
              +--------------------+
              |   REGISTER BANK    |
              |--------------------|
              | Performance Snapshot|
              | Address-Based Read |
              | Read Valid         |
              +--------------------+
                       |
                       v
                  READ INTERFACE
```

### Functional Architecture

The design consists of five major blocks.

### 3.1 Event Collector

**Module:** `event_collector.v`

The event collector receives processor-internal probe signals and generates qualified event pulses.

Responsibilities:

- Instruction retirement monitoring.
- Opcode-based instruction classification.
- Branch, load, and store event identification.
- RAW hazard monitoring.
- Load-use hazard monitoring.
- Control hazard monitoring.
- Stall, bubble, and flush detection.
- Forwarding opportunity and success monitoring.
- Pipeline stage activity monitoring.
- Privilege-mode filtering.
- Execution-context filtering.

The module registers incoming probe signals to provide a timing boundary between the processor and monitoring subsystem.

### 3.2 Mode Controller

**Module:** `mode_controller.v`

The mode controller selects the monitoring configuration.

| Mode | Architectural Counters | Microarchitectural Counters |
|---|---|---|
| Architectural Profiling | Enabled | Disabled |
| Microarchitectural Profiling | Enabled | Enabled |

Architectural profiling captures fundamental processor execution statistics.

Microarchitectural profiling additionally enables detailed pipeline and hazard monitoring.

This separation allows the monitoring architecture to selectively collect detailed microarchitectural events.

### 3.3 Event Counter

**Module:** `event_counter.v`

The event counter accumulates qualified events generated by the event collector.

Implemented capabilities:

- Parameterized counter width.
- Cycle counting.
- Instruction retirement counting.
- Branch counting.
- Load and store counting.
- RAW, load-use, and control hazard counting.
- Stall, bubble, and flush counting.
- Forwarding event counting.
- Pipeline stage activity counting.
- Counter enable and clear controls.
- Optional counter saturation to prevent rollover.

The instruction retirement counter supports configurable issue width.

### 3.4 Analytics Engine

**Module:** `analytics_engine.v`

The analytics engine converts accumulated event counters into meaningful processor performance indicators.

The calculations are implemented using synthesizable RTL arithmetic and registered metric outputs.

The metrics are represented using integer fixed-point values scaled by 100.

For example:

`125` represents `1.25`.

The engine includes division-by-zero protection.

### 3.5 Register Bank

**Module:** `register_bank.v`

The register bank stores the performance information for external readout.

Features:

- Parameterized register count.
- Parameterized data width.
- Address-based register selection.
- Read enable.
- Read data output.
- Read valid indication.
- Snapshot freeze after execution completion.
- Invalid address protection.

The standalone analytics architecture provides a 32-word register interface.

---

## 4. Monitored Performance Events

The IP monitors the following categories of processor events.

| Category | Performance Counter |
|---|---|
| Execution | Total Cycle Count |
| Execution | Instructions Retired |
| Control Flow | Branch Count |
| Memory | Load Count |
| Memory | Store Count |
| Data Hazards | RAW Hazard Count |
| Data Hazards | Load-Use Hazard Count |
| Control Hazards | Control Hazard Count |
| Pipeline | Stall Count |
| Pipeline | Bubble Count |
| Pipeline | Flush Count |
| Forwarding | Forwarding Opportunities |
| Forwarding | Forwarding Successes |
| Pipeline Activity | IF Active Cycles |
| Pipeline Activity | ID Active Cycles |
| Pipeline Activity | EX Active Cycles |
| Pipeline Activity | MEM Active Cycles |
| Pipeline Activity | WB Active Cycles |

Event counts depend on the timing and semantics of the processor probe signals connected to the analytics IP.

---

## 5. Performance Metrics

The analytics engine calculates the following performance indicators.

### 5.1 Instructions Per Cycle (IPC)

Measures the average number of instructions retired per clock cycle.

\[
IPC=\frac{\text{Instructions Retired}}{\text{Total Cycles}}
\]

A higher IPC generally indicates better instruction throughput.

### 5.2 Cycles Per Instruction (CPI)

Measures the average number of clock cycles required per retired instruction.

\[
CPI=\frac{\text{Total Cycles}}{\text{Instructions Retired}}
\]

A lower CPI generally indicates better execution efficiency.

### 5.3 Pipeline Efficiency

Measures the utilization of the configured instruction issue capacity.

\[
Efficiency=
\frac{Instructions\ Retired}
{Cycles\times Issue\ Width}\times100
\]

### 5.4 Pipeline Stage Utilization

Measures the percentage of cycles during which each pipeline stage is active.

\[
U_{stage}=
\frac{Stage\ Active\ Cycles}{Total\ Cycles}\times100
\]

The design calculates utilization independently for:

- Instruction Fetch (IF)
- Instruction Decode (ID)
- Execute (EX)
- Memory (MEM)
- Writeback (WB)

### 5.5 Forwarding Efficiency

The analytics engine calculates forwarding efficiency using the forwarding count supplied to it and the RAW hazard count.

\[
Forwarding\ Efficiency=
\frac{Forwarding\ Count}
{Forwarding\ Count+RAW\ Hazards}\times100
\]

Note: In the current top-level implementation, the analytics engine receives the forwarding-success counter as its forwarding-count input.

### 5.6 Hazard Distribution

Calculates the relative contribution of individual hazard categories.

\[
Hazard\ Distribution=
\frac{Individual\ Hazard\ Count}
{Total\ Hazard\ Count}\times100
\]

The engine calculates separate distributions for:

- RAW hazards.
- Load-use hazards.
- Control hazards.

### 5.7 Memory Operation Ratio

Measures the proportion of load and store operations relative to retired instructions.

\[
Memory\ Ratio=
\frac{Loads+Stores}{Instructions\ Retired}\times100
\]

---

## 6. Register Map

The standalone analytics top-level packs the performance counters and calculated metrics into a 32-word register bank.

| Address | Register | Description |
|---|---|---|
| 0x00 | Cycle Count | Total monitored cycles |
| 0x01 | Instructions Retired | Retired instruction count |
| 0x02 | Branch Count | Retired branches |
| 0x03 | Load Count | Load operations |
| 0x04 | Store Count | Store operations |
| 0x05 | RAW Hazard Count | RAW hazards |
| 0x06 | Load Hazard Count | Load-use hazards |
| 0x07 | Control Hazard Count | Control hazards |
| 0x08 | Stall Count | Pipeline stalls |
| 0x09 | Bubble Count | Pipeline bubbles |
| 0x0A | Flush Count | Pipeline flush events |
| 0x0B | Forwarding Opportunities | Forwarding opportunities |
| 0x0C | Forwarding Successes | Successful forwarding events |
| 0x0D | IF Active Cycles | Fetch activity |
| 0x0E | ID Active Cycles | Decode activity |
| 0x0F | EX Active Cycles | Execute activity |
| 0x10 | MEM Active Cycles | Memory activity |
| 0x11 | WB Active Cycles | Writeback activity |
| 0x12 | IPC | Instructions per cycle |
| 0x13 | CPI | Cycles per instruction |
| 0x14 | Pipeline Efficiency | Overall pipeline efficiency |
| 0x15 | IF Utilization | Fetch utilization |
| 0x16 | ID Utilization | Decode utilization |
| 0x17 | EX Utilization | Execute utilization |
| 0x18 | MEM Utilization | Memory utilization |
| 0x19 | WB Utilization | Writeback utilization |
| 0x1A | Forwarding Efficiency | Forwarding metric |
| 0x1B | RAW Hazard Distribution | RAW hazard percentage |
| 0x1C | Load Hazard Distribution | Load-use hazard percentage |
| 0x1D | Control Hazard Distribution | Control hazard percentage |
| 0x1E | Memory Ratio | Memory operation percentage |
| 0x1F | Reserved | Unused register |

Metric registers use fixed-point representation scaled by 100.

---

## 7. Processor Integration

The repository also contains a processor implementation with an integrated analytics wrapper.

**Processor directory:** `processor_0.1/`

The processor design includes:

- Program Counter.
- Instruction Memory.
- Instruction Decoder.
- Register File.
- ALU.
- Immediate Generator.
- Branch Condition Logic.
- Hazard Unit.
- Data Memory.
- CSR Register Logic.
- Timer.
- Multiplexer Logic.

### Processor Analytics Wrapper

**Module:** `processor_analytics_wrapper.v`

This module connects the processor and analytics subsystem.

It maps processor-internal signals to the analytics probe interface, including:

- Instruction commit.
- Opcode.
- Pipeline stalls.
- Pipeline flushes.
- Forwarding signals.
- Memory read/write controls.
- Pipeline stage activity.
- Privilege and context information.

### FPGA Top-Level

**Module:** `top_fpga.v`

Provides the processor-integrated top-level interface:

- Clock.
- Reset.
- Timer interrupt.
- Register read address.
- Read enable.
- Read data.
- Read valid.

The processor-integrated wrapper currently uses fixed monitoring configuration assignments. Some probe signals are approximations derived from existing processor control signals; their event semantics should be validated before using the resulting measurements as definitive processor performance results.

---

## 8. Repository Structure

```text
Config_Processor_Analytic/
│
├── Analytic_design_and_tb/
│   │
│   ├── analytics_engine.v
│   ├── analytics_top.v
│   ├── event_collector.v
│   ├── event_counter.v
│   ├── mode_controller.v
│   ├── register_bank.v
│   │
│   ├── tb_analytics_engine.v
│   ├── tb_event_collector.v
│   ├── tb_event_counter.v
│   ├── tb_mode_controller.v
│   ├── tb_register_bank.v
│   └── tb_top.v
│
├── processor_0.1/
│   │
│   ├── processor_converted.v
│   ├── processor_analytics_wrapper.v
│   ├── top_fpga.v
│   ├── alu_converted.v
│   ├── br_cond_converted.v
│   ├── controller_converted.v
│   ├── csr_reg_converted.v
│   ├── data_mem_converted.v
│   ├── hazard_unit_converted.v
│   ├── imm_gen_converted.v
│   ├── inst_dec_converted.v
│   ├── inst_mem_converted.v
│   ├── mux_2x1_converted.v
│   ├── mux_4x1_converted.v
│   ├── pc_converted.v
│   ├── reg_file_converted.v
│   ├── timer_converted.v
│   └── Memory initialization files
│
├── tb_processor_only.v
│
├── Configurable_Processor_Analytics_IP_v1.0.docx
├── capable_to_produce.docx
│
└── paper and links/
    └── Research papers
```

---

## 9. Verification and Testing

The project includes dedicated testbenches for individual analytics modules and complete subsystem integration.

| Testbench | Verification Objective |
|---|---|
| tb_event_collector.v | Event detection and probe qualification |
| tb_event_counter.v | Counter accumulation, reset, clear, and saturation |
| tb_analytics_engine.v | Performance metric calculations |
| tb_mode_controller.v | Profiling mode selection |
| tb_register_bank.v | Register snapshot and read operations |
| tb_top.v | Complete analytics subsystem verification |
| tb_processor_only.v | Processor instruction execution and functional verification |

### Verification Approach

The verification process covers:

1. Reset and initialization behavior.
2. Counter enable and clear functionality.
3. Architectural and microarchitectural profiling.
4. Instruction classification.
5. Hazard and forwarding event generation.
6. Pipeline activity monitoring.
7. Metric calculation and fixed-point scaling.
8. Division-by-zero protection.
9. Register address decoding.
10. Snapshot capture and readout.
11. Processor instruction execution.
12. Integration between processor and analytics subsystem.

The processor testbench also documents known functional issues in certain instruction and CSR operations. These should be addressed and regression-tested before claiming full processor verification.

---

## 10. Simulation

The RTL is written in Verilog and can be simulated using compatible HDL simulators.

Supported simulation options include:

- Icarus Verilog.
- ModelSim / Questa.
- Cadence Xcelium.

### Icarus Verilog Example

Clone the repository:

```bash
git clone https://github.com/PJ-tech-dev/Config_Processor_Analytic.git

cd Config_Processor_Analytic
```

Compile the standalone analytics testbench:

```bash
iverilog -g2001 -s tb_analytics_top -o analytics_sim \
Analytic_design_and_tb/event_collector.v \
Analytic_design_and_tb/event_counter.v \
Analytic_design_and_tb/mode_controller.v \
Analytic_design_and_tb/register_bank.v \
Analytic_design_and_tb/analytics_engine.v \
Analytic_design_and_tb/analytics_top.v \
Analytic_design_and_tb/tb_top.v
```

Run simulation:

```bash
vvp analytics_sim
```

Inspect the simulation output for counter values, calculated metrics, and testbench pass/fail results.

The processor-only testbench is a separate simulation target and requires the processor RTL modules.

---

## 11. Design Considerations

The following considerations are important when evaluating or extending the IP:

- **Event accuracy:** Correct performance measurements depend on the accuracy of processor probe signals.
- **Counter sizing:** Counter width must accommodate the expected execution duration.
- **Metric precision:** Metric outputs use integer fixed-point representation.
- **Pipeline monitoring:** Valid and stall signals must represent the intended stage activity.
- **Clock domain:** Processor and analytics clock domains must be compatible or use appropriate CDC logic.
- **Snapshot behavior:** The current register bank freezes the captured snapshot after execution completion until reset.
- **Hardware cost:** Division and multiplication in the analytics engine should be evaluated for area and timing on the target FPGA or ASIC library.
- **Integration verification:** Standalone analytics verification and processor-integrated verification must be performed independently.

---

## 12. Future Enhancements

Potential extensions include:

- APB or AXI-Lite register interface.
- Software-accessible performance monitoring registers.
- Programmable event selection.
- Configurable performance thresholds.
- Interrupt generation based on performance events.
- Cache and memory hierarchy monitoring.
- Branch prediction performance analysis.
- Multi-issue processor support.
- Per-context performance profiling.
- Power-aware event monitoring.
- SVA-based verification.
- Automated performance report generation.

These are proposed extensions and are not represented as completed functionality.

---

## 13. Technologies Used

| Category | Technology |
|---|---|
| Hardware Description | Verilog HDL |
| Design Methodology | Modular RTL Design |
| Processor Architecture | RISC-V |
| Monitoring Architecture | Hardware Performance Counters |
| Verification | Verilog Testbenches |
| Simulation | HDL Simulation Tools |
| Target Applications | FPGA / ASIC-oriented RTL |

---

## 14. Applications

- Processor performance characterization.
- Pipeline bottleneck identification.
- Hardware performance monitoring.
- Architectural and microarchitectural analysis.
- Processor RTL verification.
- FPGA-based processor analytics.
- Research in configurable processor monitoring IP.

---

## 15. Getting Started

To explore the implementation:

1. Start with `event_collector.v` to understand processor event qualification.
2. Examine `event_counter.v` to understand hardware counter accumulation.
3. Study `mode_controller.v` to understand profiling configurations.
4. Inspect `analytics_engine.v` to understand metric computation.
5. Review `register_bank.v` to understand performance data storage.
6. Examine `analytics_top.v` for complete subsystem integration.
7. Open `processor_analytics_wrapper.v` to understand processor integration.

---

## 16. Author

**Project:** Configurable Processor Performance Monitoring and Analytics IP

**Developer:** Pranav J

**GitHub:** https://github.com/PJ-tech-dev

**Repository:** https://github.com/PJ-tech-dev/Config_Processor_Analytic

---

## 17. Research References and Documentation

Supporting documents and research papers are available in the repository:

- `Configurable_Processor_Analytics_IP_v1.0.docx`
- `capable_to_produce.docx`
- `paper and links/`

These materials provide additional technical background and research context for the analytics architecture.

---

<p align="center">
  <strong>Configurable Processor Performance Monitoring and Analytics IP</strong>
  <br>
  Hardware-based visibility into processor execution and performance.
</p>
