# 5-Stage Pipelined RISC-V (RV32I) Processor — VHDL Implementation

**Course:** EEE413/ECE413 Digital System Design, BRAC University
**Group:** Group 4, Section 02
**Members:** Sujana Haque (22221007), Intisar Ahmed (22221111), Tanvir Jubaer (22321052), Satirtha Saha (22321058), Mahdi Abrar Yousuf (22321069)

A classic 5-stage (IF/ID/EX/MEM/WB) pipelined RV32I core, implemented in two functionally-equivalent variants that resolve branches at different pipeline stages, plus a minimal worked-example configuration used for a hand-traceable walkthrough. Verified in GHDL, simulated in Xilinx ISim, and synthesized on Xilinx ISE targeting an Artix-7 FPGA.

This repository contains source code, testbenches, tooling, and verification/synthesis results only. The written report and slide deck are maintained separately and are not included here.

---

## Architecture

Standard 5-stage pipeline: **IF → ID → EX → MEM → WB**, with the usual pipeline registers (`if_id_reg`, `id_ex_reg`, `ex_mem_reg`, `mem_wb_reg`), a register file with write-through same-cycle bypass, full EX/MEM and MEM/WB operand forwarding, load-use hazard detection/stalling, and branch-flush control.

Two core variants share this structure but differ in **where branches/jumps are resolved**:

| Variant | Top entity | Branch/jump resolved in | Extra units needed |
|---|---|---|---|
| EX-stage resolution | `cpu_top.vhd` / `datapath.vhd` | EX | `forwarding_unit.vhd`, `hazard_detection_unit.vhd` |
| ID-stage resolution | `cpu_top_idres.vhd` / `datapath_idres.vhd` | ID | + `id_forward_unit.vhd`, `hazard_detection_unit_idres.vhd` |

Both pass an identical 26-register architectural correctness check; they trade cycle count, LUT usage, and Fmax against each other (see `results/cycle_count_comparison.md` and `results/synthesis_results.md`).

A third, minimal configuration — `cpu_top_worked.vhd` / `datapath_worked.vhd` / `instr_mem_worked.vhd` — runs a deliberately tiny 3-instruction program and exists to support a fully hand-traceable worked example, not as a third "real" variant.

### Shared building blocks (`src/`)
`riscv_pkg.vhd` (shared types/constants), `pc_reg.vhd`, `register_file.vhd`, `immediate_gen.vhd`, `alu.vhd` + `alu_control.vhd`, `branch_unit.vhd`, `control_unit.vhd`, `load_store_unit.vhd`, `data_mem.vhd`, `instr_mem.vhd`, and the four pipeline registers.

---

## Repository layout

```
src/      26 synthesizable VHDL source files (shared units + all three top-level variants)
tb/       6 self-checking testbenches
tools/    asm_full.py -- custom RV32I assembler used to build the test program(s)
results/  verification results, synthesis reports, RTL schematics, ISim waveforms (the "findings")
docs/     design notes, bugs found, known limitations
```

## Toolchain

| Purpose | Tool |
|---|---|
| Simulation (regression) | GHDL 4.1.0 (mcode backend, `--std=93 -fsynopsys`) |
| Simulation (waveforms) | Xilinx ISim |
| Synthesis | Xilinx ISE 14.7 / XST |
| Target device | xc7a100t-3csg324 (Artix-7) |
| Test program assembly | Custom Python assembler (`tools/asm_full.py`) |

## Running the testbenches (GHDL)

The `-fsynopsys` flag is required because these testbenches use `std_logic_textio` for trace logging.

```bash
cd src
ghdl -a --std=93 riscv_pkg.vhd pc_reg.vhd if_id_reg.vhd id_ex_reg.vhd ex_mem_reg.vhd \
        mem_wb_reg.vhd register_file.vhd immediate_gen.vhd alu.vhd alu_control.vhd \
        branch_unit.vhd control_unit.vhd forwarding_unit.vhd id_forward_unit.vhd \
        hazard_detection_unit.vhd hazard_detection_unit_idres.vhd load_store_unit.vhd \
        data_mem.vhd instr_mem.vhd datapath.vhd cpu_top.vhd datapath_idres.vhd \
        cpu_top_idres.vhd instr_mem_worked.vhd datapath_worked.vhd cpu_top_worked.vhd

cd ../tb
ghdl -a --std=93 -fsynopsys tb_alu.vhd
ghdl -e --std=93 -fsynopsys tb_alu
ghdl -r --std=93 -fsynopsys tb_alu
# repeat -a/-e/-r for tb_register_file, tb_hazard_forwarding, tb_cpu_top,
# tb_cpu_top_idres, tb_worked_example
```

See [`results/testbench_results.md`](results/testbench_results.md) for expected output (93/93 assertions passing across all 6 testbenches).

---

## Status

- [`results/testbench_results.md`](results/testbench_results.md) — full verification results (freshly re-run under GHDL 4.1.0).
- [`results/synthesis_results.md`](results/synthesis_results.md) — real XST device utilization and Fmax for both CPU variants.
- [`results/cycle_count_comparison.md`](results/cycle_count_comparison.md) — cycle-count trade-off between EX-stage and ID-stage branch resolution.
- [`results/schematics/`](results/schematics/) — real RTL/Technology schematic screenshots for every submodule.
- [`results/waveforms/`](results/waveforms/) — real ISim waveform screenshots plus raw `.vcd` dumps.
- [`docs/findings.md`](docs/findings.md) — bugs found during simulation, key design decisions, and known limitations.
