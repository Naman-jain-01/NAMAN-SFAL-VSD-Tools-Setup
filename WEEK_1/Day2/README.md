```markdown
# D Flip-Flop Modules with Asynchronous and Synchronous Control

This repository contains Verilog modules implementing D flip-flops (DFFs) with different set and reset configurations: asynchronous set, asynchronous reset, and combined asynchronous/synchronous reset. Below is a detailed explanation of each module, their functionality, differences, and synthesis insights, formatted for clarity and understanding.

## Table of Contents
- [Overview](#overview)
- [Module Descriptions](#module-descriptions)
  - [1. `dff_async_set`](#1-dff_async_set)
  - [2. `dff_asyncres`](#2-dff_asyncres)
  - [3. Synthesized Netlist for `dff_asyncres`](#3-synthesized-netlist-for-dff_asyncres)
  - [4. `dff_asyncres_syncres`](#4-dff_asyncres_syncres)
- [Key Differences](#key-differences)
- [Use Cases](#use-cases)
- [Synthesis Considerations](#synthesis-considerations)
- [License](#license)

## Overview
The modules implement D flip-flops with varying control mechanisms:
- **`dff_async_set`**: DFF with an asynchronous active-high set.
- **`dff_asyncres`**: DFF with an asynchronous active-high reset.
- **`dff_asyncres_syncres`**: DFF with both asynchronous and synchronous active-high resets.

Each module is written in Verilog, with a synthesized netlist provided for `dff_asyncres` to illustrate hardware mapping using the Sky130 process design kit (PDK).

## Module Descriptions

### 1. `dff_async_set`
**Code:**
```verilog
module dff_async_set ( input clk, input async_set, input d, output reg q );
always @ (posedge clk, posedge async_set)
begin
    if (async_set)
        q <= 1'b1;
    else
        q <= d;
end
endmodule
```

**Functionality:**
- **Purpose**: D flip-flop with an asynchronous active-high set input.
- **Inputs**:
  - `clk`: Clock signal (positive edge triggers data transfer).
  - `async_set`: Asynchronous set signal (sets `q` to 1 when high).
  - `d`: Data input.
- **Output**: `q` (flip-flop output).
- **Behavior**:
  - When `async_set = 1`, `q` is immediately set to `1'b1` (independent of clock).
  - When `async_set = 0`, `q` updates to `d` on the positive edge of `clk`.
- **Key Feature**: Asynchronous set takes precedence over clocked data transfer.

### 2. `dff_asyncres`
**Code:**
```verilog
module dff_asyncres ( input clk, input async_reset, input d, output reg q );
always @ (posedge clk, posedge async_reset)
begin
    if (async_reset)
        q <= 1'b0;
    else
        q <= d;
end
endmodule
```

**Functionality:**
- **Purpose**: D flip-flop with an asynchronous active-high reset input.
- **Inputs**:
  - `clk`: Clock signal (positive edge triggers data transfer).
  - `async_reset`: Asynchronous reset signal (resets `q` to 0 when high).
  - `d`: Data input.
- **Output**: `q` (flip-flop output).
- **Behavior**:
  - When `async_reset = 1`, `q` is immediately reset to `1'b0` (independent of clock).
  - When `async_reset = 0`, `q` updates to `d` on the positive edge of `clk`.
- **Key Feature**: Asynchronous reset takes precedence over clocked data transfer.

### 3. Synthesized Netlist for `dff_asyncres`
**Code:**
```verilog
module dff_asyncres(clk, async_reset, d, q);
  wire _0_;
  wire _1_;
  wire _2_;
  input async_reset;
  input clk;
  input d;
  output q;
  sky130_fd_sc_hd__clkinv_1 _3_ (
    .A(_0_),
    .Y(_1_)
  );
  sky130_fd_sc_hd__dfrtp_1 _4_ (
    .CLK(clk),
    .D(d),
    .Q(q),
    .RESET_B(_2_)
  );
  assign _0_ = async_reset;
  assign _2_ = _1_;
endmodule
```

**Explanation:**
- **Purpose**: Synthesized hardware implementation of `dff_asyncres` using Sky130 PDK.
- **Components**:
  - `sky130_fd_sc_hd__clkinv_1`: Inverter to convert active-high `async_reset` to active-low for the flip-flop.
  - `sky130_fd_sc_hd__dfrtp_1`: D flip-flop with active-low reset (`RESET_B`).
- **Behavior**:
  - The inverter converts `async_reset` (active-high) to an active-low signal for the flip-flop’s `RESET_B` input.
  - When `async_reset = 1`, the flip-flop resets `q` to 0; otherwise, `q` updates to `d` on the clock’s positive edge.
- **Key Insight**: Synthesis adapts the active-high reset to the active-low reset of the library cell using an inverter.

### 4. `dff_asyncres_syncres`
**Code:**
```verilog
module dff_asyncres_syncres ( input clk, input async_reset, input sync_reset, input d, output reg q );
always @ (posedge clk, posedge async_reset)
begin
    if (async_reset)
        q <= 1'b0;
    else if (sync_reset)
        q <= 1'b0;
    else
        q <= d;
end
endmodule
```

**Functionality:**
- **Purpose**: D flip-flop with both asynchronous and synchronous active-high resets.
- **Inputs**:
  - `clk`: Clock signal (positive edge triggers data transfer and synchronous reset).
  - `async_reset`: Asynchronous reset signal (resets `q` to 0 when high).
  - `sync_reset`: Synchronous reset signal (resets `q` to 0 on clock edge when high).
  - `d`: Data input.
- **Output**: `q` (flip-flop output).
- **Behavior**:
  - When `async_reset = 1`, `q` is immediately reset to `1'b0` (independent of clock).
  - When `async_reset = 0` and `sync_reset = 1`, `q` is reset to `1'b0` on the positive edge of `clk`.
  - When `async_reset = 0` and `sync_reset = 0`, `q` updates to `d` on the positive edge of `clk`.
- **Key Feature**: Combines asynchronous and synchronous resets with priority: async reset > sync reset > data transfer.

## Key Differences

| Feature                   | `dff_async_set`                          | `dff_asyncres`                          | `dff_asyncres_syncres`                          |
|---------------------------|------------------------------------------|-----------------------------------------|-------------------------------------------------|
| **Purpose**               | Async set DFF                           | Async reset DFF                        | Async + sync reset DFF                         |
| **Inputs**                | `clk`, `async_set`, `d`                 | `clk`, `async_reset`, `d`              | `clk`, `async_reset`, `sync_reset`, `d`        |
| **Output**                | `q`                                     | `q`                                    | `q`                                            |
| **Control Action**        | Sets `q` to 1 (async)                   | Resets `q` to 0 (async)                | Resets `q` to 0 (async or sync)                |
| **Synchronous Reset**     | No                                      | No                                     | Yes (`sync_reset`)                             |
| **Priority**              | Async set > Data transfer               | Async reset > Data transfer            | Async reset > Sync reset > Data transfer       |
| **Synthesis**             | Requires preset-capable FF              | Uses inverter + active-low reset FF     | Requires async reset FF + sync reset logic      |

**Summary**:
- **`dff_async_set`**: Forces `q` to 1 asynchronously (set operation).
- **`dff_asyncres`**: Forces `q` to 0 asynchronously (reset operation).
- **`dff_asyncres_syncres`**: Supports both async and sync resets, offering flexibility for complex designs.

## Use Cases
- **`dff_async_set`**: Initialize registers to 1 (e.g., enabling signals, counters starting at 1).
- **`dff_asyncres`**: Initialize registers to 0 (e.g., clearing state machines, resetting counters).
- **`dff_asyncres_syncres`**: Combine immediate (async) and controlled (sync) resets (e.g., power-on reset + operational reset in state machines).

## Synthesis Considerations
- **Asynchronous Controls**: Async set/reset signals (`async_set`, `async_reset`) bypass the clock, requiring careful glitch management to avoid metastability.
- **Synchronous Reset**: `sync_reset` in `dff_asyncres_syncres` aligns with the clock, reducing timing issues but adding logic complexity.
- **Netlist Insight**: The `dff_asyncres` netlist shows how synthesis tools adapt active-high resets to active-low library cells using inverters, a common practice in ASIC design.
- **Resource Usage**: `dff_asyncres_syncres` may require additional logic (e.g., multiplexers) for synchronous reset, increasing area compared to the other modules.
```