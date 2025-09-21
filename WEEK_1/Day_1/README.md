```markdown
# Open-Source RTL Design and Synthesis Workflow with Sky130

This README provides a comprehensive guide to using open-source tools for RTL (Register-Transfer Level) design, simulation, synthesis, and verification using the Sky130 PDK (Process Design Kit). We focus on:

- **Icarus Verilog (iverilog)**: Verilog simulator for compiling and running RTL code.
- **GTKWave**: Waveform viewer for analyzing simulation outputs (VCD files).
- **Multiplexer Circuit**: A practical example of a 4:1 multiplexer in Verilog, including testbench, simulation, and synthesis.
- **Yosys**: RTL synthesizer for generating gate-level netlists.
- **Synthesis with Sky130**: Mapping the design to Sky130 standard cells using Yosys and the Sky130 liberty library.

This workflow is ideal for learning ASIC/FPGA design flows, especially for Tiny Tapeout or open-source silicon projects. All tools are free and open-source.

**Note**: Assumes a Linux environment (Ubuntu/Debian recommended). Installation scripts and examples are provided.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Tool Installation](#tool-installation)
- [Icarus Verilog (iverilog) Usage](#icarus-verilog-iverilog-usage)
- [GTKWave Usage](#gtkwave-usage)
- [Multiplexer Circuit Example](#multiplexer-circuit-example)
  - [Verilog Code](#verilog-code)
  - [Testbench](#testbench)
  - [Simulation](#simulation)
  - [Waveform Viewing](#waveform-viewing)
- [Yosys Synthesis](#yosys-synthesis)
- [Synthesis with Sky130](#synthesis-with-sky130)
- [Gate-Level Simulation (GLS)](#gate-level-simulation-gls)
- [Troubleshooting](#troubleshooting)
- [References](#references)
- [License](#license)

## Prerequisites
- Linux OS (e.g., Ubuntu 20.04+).
- Basic Verilog knowledge.
- Git for cloning repositories.
- Download the Sky130 PDK and liberty files from the [Google SkyWater PDK GitHub](https://github.com/google/skywater-pdk) or use a pre-built setup like VSDFlow.

## Tool Installation
Install tools via Conda (recommended for isolation) or system packages.

### Using Conda (litex-hub Channel)
```bash
# Install Miniconda if not present: https://docs.conda.io/en/latest/miniconda.html
conda config --add channels litex-hub
conda config --add channels conda-forge
conda create -n sky130 yosys iverilog gtkwave open_pdks.sky130a
conda activate sky130

# Install Sky130 PDK
conda install -c litex-hub open_pdks.sky130a=1.0.457_0_g32e8f23

# Clone example repos (optional)
git clone https://github.com/YosysHQ/yosys.git
git clone https://github.com/steveicarus/iverilog.git  # For building if needed
```

### System Packages (Ubuntu)
```bash
sudo apt update
sudo apt install iverilog gtkwave yosys
# For Sky130, download manually: https://github.com/google/skywater-pdk/releases
# Extract to ~/sky130/ and set env: export PDK_ROOT=~/sky130A/
```

Verify installations:
```bash
iverilog -v  # Should show version
gtkwave -v   # Should show version
yosys -V     # Should show version
```

## Icarus Verilog (iverilog) Usage
Icarus Verilog compiles Verilog code into a simulation executable (.vvp) and runs it with `vvp`.

### Basic Steps
1. Write Verilog module (e.g., `module.v`).
2. Write testbench (e.g., `tb.v`).
3. Compile: `iverilog -o sim.vvp module.v tb.v`
4. Run: `vvp sim.vvp` (dumps VCD if `$dumpfile` used in testbench).
5. View: `gtkwave dump.vcd`

### Example Command
```bash
iverilog -o mux.vvp mux.v tb_mux.v
vvp mux.vvp
```

**Tips**:
- Use `-Wall` for warnings: `iverilog -Wall -o sim.vvp *.v`
- For large designs: Use `-g2012` for SystemVerilog support.
- Output: Console logs via `$display/$monitor`; waveforms via `$dumpvars`.

## GTKWave Usage
GTKWave visualizes VCD/FST waveform dumps from simulations.

### Basic Steps
1. Run simulation to generate `dump.vcd`.
2. Launch: `gtkwave dump.vcd`
3. In GUI:
   - **Hierarchy**: Browse design hierarchy (left panel).
   - **Signals**: Drag signals to the waveform window.
   - **Waves**: Zoom (Ctrl+Mouse Wheel), search signals (Ctrl+F).
   - **Markers**: Add cursors for timing analysis.
   - **Save Session**: File > Write Save File As > `session.gtkw` for reuse.
4. Reload: Ctrl+R (reloads VCD without closing).

### Advanced Features
- **Filters**: Use regex in signal search (e.g., `clk*` for clock signals).
- **Colors**: Right-click signal > Colors > Set custom colors.
- **Export**: File > Export > Image for screenshots.
- Command Line: `gtkwave -a session.gtkw dump.vcd` (load save file).

**Tips**:
- For large VCDs (>1GB), use FST format: Add `-fst` to `vvp` (faster loading).
- Keyboard Shortcuts: Space (zoom fit), T (toggle transaction view).

## Multiplexer Circuit Example
We'll design a 4:1 multiplexer (MUX): Selects one of four 4-bit inputs based on a 2-bit select signal.

### Verilog Code
**mux_4x1.v** (Behavioral Model):
```verilog
module mux_4x1 (
    input [3:0] a, b, c, d,  // 4-bit inputs
    input [1:0] sel,         // 2-bit select
    output reg [3:0] y       // 4-bit output
);
    always @(*) begin
        case (sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
            2'b11: y = d;
            default: y = 4'bxxxx;
        endcase
    end
endmodule
```

Alternative (Dataflow):
```verilog
assign y = (sel == 2'b00) ? a :
           (sel == 2'b01) ? b :
           (sel == 2'b10) ? c : d;
```

### Testbench
**tb_mux.v**:
```verilog
module tb_mux;
    reg [3:0] a, b, c, d;
    reg [1:0] sel;
    wire [3:0] y;

    mux_4x1 dut (.a(a), .b(b), .c(c), .d(d), .sel(sel), .y(y));

    initial begin
        $dumpfile("mux.vcd");
        $dumpvars(0, tb_mux);

        a = 4'h1; b = 4'h2; c = 4'h3; d = 4'h4;
        sel = 2'b00; #10; $display("sel=00, y=%h", y);  // Expected: 1
        sel = 2'b01; #10; $display("sel=01, y=%h", y);  // Expected: 2
        sel = 2'b10; #10; $display("sel=10, y=%h", y);  // Expected: 3
        sel = 2'b11; #10; $display("sel=11, y=%h", y);  // Expected: 4
        $finish;
    end
endmodule
```

### Simulation
```bash
iverilog -o mux.vvp mux_4x1.v tb_mux.v
vvp mux.vvp  # Outputs: sel=00, y=1 ... etc.
# Generates mux.vcd
```

Expected Console Output:
```
sel=00, y=1
sel=01, y=2
sel=10, y=3
sel=11, y=4
```

### Waveform Viewing
```bash
gtkwave mux.vcd
```
- Drag `sel`, `a/b/c/d`, `y` to waves.
- Verify: Output `y` follows selected input at each #10 timestep.

## Yosys Synthesis
Yosys synthesizes Verilog RTL to gate-level netlist.

### Basic Steps
1. Launch: `yosys`
2. Read Liberty (for tech mapping): `read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib`
3. Read Design: `read_verilog mux_4x1.v`
4. Synthesize: `synth -top mux_4x1`
5. Optimize/Map: `opt_clean -purge; abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib`
6. View: `show` (opens dot graph in xdot or similar).
7. Write Netlist: `write_verilog -noattr mux_netlist.v`
8. Exit: `exit`

### Example Session
```
yosys> read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog mux_4x1.v
yosys> synth -top mux_4x1
yosys> dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  # If flops present
yosys> abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
yosys> write_verilog -noattr mux_netlist.v
```
- **Output**: `mux_netlist.v` (gate-level Verilog with AND/OR/INV gates from Sky130).

**Tips**:
- Hierarchical Synthesis: Synthesize sub-modules first.
- Flatten: `flatten` for single-module netlist.
- Optimization: `opt` removes unused logic.

## Synthesis with Sky130
Sky130 is a 130nm open PDK from SkyWater. Yosys maps to its standard cells (e.g., `sky130_fd_sc_hd__and2_1`).

### Setup
- Liberty File: `~/pdks/sky130A/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
- Export: `export LIB=../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`

### Full Flow for MUX
Follow Yosys steps above. The netlist will use Sky130 cells like inverters and NAND gates for the case logic.

Example Netlist Snippet (after synthesis):
```verilog
module mux_4x1 (a, b, c, d, sel, y);
  wire _0_, _1_, ...;  // Internal wires
  sky130_fd_sc_hd__and2b_1 _10_ (.A_N(sel[0]), .B(sel[1]), .Y(_0_));  // Example gate
  // ... more gates
  assign y = ...;
endmodule
```

- Verify: Run `show` for schematic; count cells (e.g., ~6-8 gates for 4:1 MUX).

### Advanced
- Constraints: Use SDC files for timing (e.g., `read_sdc constraints.sdc`).
- Tech Mapping: `techmap; dfllibmap` for flops.

## Gate-Level Simulation (GLS)
Verify netlist matches RTL:
1. Compile: `iverilog -o gls.vvp -DGLS mux_netlist.v tb_mux.v` (define GLS to use netlist).
2. Run: `vvp gls.vvp`
3. Compare VCDs in GTKWave (RTL vs. GLS should match).

## Troubleshooting
- **Icarus Errors**: Check syntax with `iverilog -t vvp *.v` (parse only).
- **GTKWave Crashes on Large VCD**: Use FST: `$dumpfile("dump.fst"); $dumpvars(0, tb);` and `vvp -fst`.
- **Yosys No Liberty**: Ensure path to `.lib` is absolute; check `ls $LIB`.
- **Sky130 Issues**: Run `pdki setup` if using open_pdks.
- **Dot/Show Fails**: Install `xdot`: `sudo apt install xdot`.

## References
- Icarus Verilog Docs: [steveicarus.github.io/iverilog](https://steveicarus.github.io/iverilog/) 
- GTKWave Guide: [gtkwave.sourceforge.net/gtkwave.pdf](https://gtkwave.sourceforge.net/gtkwave.pdf) 
- Yosys Docs: [yosys.readthedocs.io](https://yosys.readthedocs.io/) 
- Sky130 Workshop: [github.com/kambadur/sky130RTLDesignAndSynthesisWorkshop](https://github.com/kambadur/sky130RTLDesignAndSynthesisWorkshop) 
- MUX Example: [chipverify.com/verilog/verilog-4to1-mux](https://www.chipverify.com/verilog/verilog-4to1-mux) 

```