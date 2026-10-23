# 5-Stage Pipelined RISC-V (RV32I) Processor — VHDL Implementation

**Course:** EEE413/ECE413 Digital System Design, BRAC University

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

<!-- gallery:start -->
## Results and figures

Selected schematics and simulation results. The complete set (every block, per-signal waveforms and console logs) is in [`results/`](results/).

### Core architecture (EX-stage branch resolution, `cpu_top`)

<table>
<tr><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/1.%20cpu%20top.JPG" width="100%" alt="CPU top level (RTL)"><br><sub>CPU top level (RTL)</sub></td><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/2.%20datapath.JPG" width="100%" alt="Datapath (RTL)"><br><sub>Datapath (RTL)</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/3.%20alu.JPG" width="100%" alt="ALU"><br><sub>ALU</sub></td><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/6.%20control%20unit.JPG" width="100%" alt="Control unit"><br><sub>Control unit</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/9.%20forwarding%20unit.JPG" width="100%" alt="Forwarding unit"><br><sub>Forwarding unit</sub></td><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/10.%20hazard%20detection.JPG" width="100%" alt="Hazard detection unit"><br><sub>Hazard detection unit</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/schematics/cpu_top/RTL/18.%20register%20file.JPG" width="100%" alt="Register file"><br><sub>Register file</sub></td><td align="center" width="50%"><img src="results/schematics/cpu_top/Tech/1.%20cpu%20_top.JPG" width="100%" alt="CPU top level (technology schematic)"><br><sub>CPU top level (technology schematic)</sub></td></tr>
</table>

### ID-stage branch resolution variant (`cpu_top_idres`)

<table>
<tr><td align="center" width="50%"><img src="results/schematics/cpu_top_idres/RTL/1.%20cpu_top_idres.JPG" width="100%" alt="CPU top level, ID-stage resolution (RTL)"><br><sub>CPU top level, ID-stage resolution (RTL)</sub></td><td align="center" width="50%"><img src="results/schematics/cpu_top_idres/RTL/2.%20datapath_idres.JPG" width="100%" alt="Datapath, ID-stage resolution"><br><sub>Datapath, ID-stage resolution</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/schematics/cpu_top_idres/RTL/12.%20id_forwarding%20unit.JPG" width="100%" alt="ID-stage forwarding unit"><br><sub>ID-stage forwarding unit</sub></td><td align="center" width="50%"><img src="results/schematics/cpu_top_idres/RTL/10.%20Hazard%20detection%20Idres.JPG" width="100%" alt="Hazard detection, ID-stage resolution"><br><sub>Hazard detection, ID-stage resolution</sub></td></tr>
</table>

### Synthesis design summaries

<table>
<tr><td align="center" width="50%"><img src="results/schematics/design_summary/1.%20Design%20Summary%28cpu_top%29.JPG" width="100%" alt="Design summary, cpu_top"><br><sub>Design summary, cpu_top</sub></td><td align="center" width="50%"><img src="results/schematics/design_summary/3.%20Design%20Summary%28cpu_top_idres%29.JPG" width="100%" alt="Design summary, cpu_top_idres"><br><sub>Design summary, cpu_top_idres</sub></td></tr>
</table>

### Simulation waveforms

<table>
<tr><td align="center" width="50%"><img src="results/waveforms/cpu_top/1.%20Full%20diagram.JPG" width="100%" alt="cpu_top, full waveform"><br><sub>cpu_top, full waveform</sub></td><td align="center" width="50%"><img src="results/waveforms/cpu_top_idres/1.%20full%20diagram.JPG" width="100%" alt="cpu_top_idres, full waveform"><br><sub>cpu_top_idres, full waveform</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/waveforms/cpu_top/6.%20cycle%200-33.JPG" width="100%" alt="cpu_top, cycles 0-33"><br><sub>cpu_top, cycles 0-33</sub></td><td align="center" width="50%"><img src="results/waveforms/cpu_top/7.%20cycle%2034-66.JPG" width="100%" alt="cpu_top, cycles 34-66"><br><sub>cpu_top, cycles 34-66</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/waveforms/cpu_top/8.%20cycle%2067-97.JPG" width="100%" alt="cpu_top, cycles 67-97"><br><sub>cpu_top, cycles 67-97</sub></td><td align="center" width="50%"><img src="results/waveforms/cpu_top/5.%20console%201.JPG" width="100%" alt="cpu_top, console output (register check)"><br><sub>cpu_top, console output (register check)</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/waveforms/hazard_forwarding/1.%20full%20diagram.JPG" width="100%" alt="Hazard and forwarding testbench"><br><sub>Hazard and forwarding testbench</sub></td><td align="center" width="50%"><img src="results/waveforms/alu/1.%20full%20diagram.JPG" width="100%" alt="ALU testbench"><br><sub>ALU testbench</sub></td></tr>
<tr><td align="center" width="50%"><img src="results/waveforms/register_file/1.%20full%20diagram.JPG" width="100%" alt="Register file testbench"><br><sub>Register file testbench</sub></td><td></td></tr>
</table>

<!-- gallery:end -->

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

## Contributors

Sujana, Intisar, Tanvir, Satirtha and Mahdi.

## License

Released under the MIT License. See [`LICENSE`](LICENSE).
