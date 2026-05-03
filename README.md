# RTL Design & Synthesis Workshop

Internship screening covering Verilog RTL design, logic synthesis, and flop coding styles using EDA tools.

---

## Module 1 – Introduction to Verilog RTL Design & Synthesis

Here, we begin with a brief introduction to RTL coding. Explaining how it serves as an interpretation of our abstract logic. We have leant how to simulate using iverilog, view waveforms in GTKWave and perform synthesis using Yosys.

### Step 1 – Simulate with iverilog

```bash
sudo apt install iverilog gtkwave
iverilog -o sim design.v testbench.v
./sim
```

![iverilog simulation](result_images/task1.png)

### Step 2 – View waveforms in GTKWave
The functionality and timing behavior of our module can be monitored using testbenches (or tb files). Here we examined a simple 2 x 1 mux.

```bash
gtkwave dump.vcd
```

![GTKWave waveform](result_images/task2.png)

### Step 3 – Synthesise with Yosys
Synthesis maps the abstract hardware logic to cells, this is done by generating netlists.
```bash
yosys
> read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
> read_verilog good_mux.v
> synth -top good_mux
```

![Yosys initialisation](result_images/yosysinitialisation.png)

### Step 4 – Map to standard cells 
Yosys enables us to examine netlists and leaf cells vua the following few commands:

```bash
> abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
> show
> write_verilog -noattr netlist.v
```

![Generated netlist](result_images/generated_netlist.png)
![Corrected schematic](result_images/corrected_sch.png)

### Step 5 – Verify against Sky130 PDKs

Re-run the testbench on the synthesised netlist to confirm it matches RTL simulation.

![Verification](result_images/inputandoutputyosys.png)

---

## Module 2 – Timing Libs, Hierarchical vs Flat Synthesis & Flop Coding Styles

### Step 6 – Exploring libraries

Open the Sky130 `.lib` file and study cell attributes: timing arcs, drive strength, and power. We also further employ a multple_module.v file to examine the two main kinds of synthesis.

```bash
gvim sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Step 7 – Hierarchical vs Flat Synthesis

**Hierarchical** – As the name suggests, the hierarchy of the submodules are maintained and the schematic does not display the internal workings of our module:

```bash
> synth -top multiple_modules
> show multiple_modules
```

![Hierarchy visible](result_images/hierarchy_visible.png)

**Flat** – all hierarchy is dissolved and the internal functioning of the modules are displayed when the "show" command is used:

```bash
> flatten
> show
```

![Flattened netlist](result_images/flattened+netlist.png)
![Flattened schematic](result_images/flattened_sch.png)

**Sub-module synthesis** – Primarily useful for large or repeated blocks:

```bash
> synth -top sub_module1
```

![Sub-module](result_images/submodule1.png)

### Step 8 – Flop Coding Styles

Simulate synthesise and examine different D flip-flop variants. Run `dfflibmap` before ABC to pick the correct flip-flop cells. The primary flip flops examined were those with asynchronus and synchronous resets: 

```bash
> dfflibmap -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
> abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
> show
```

| Style | Simulation | Schematic |
|-------|-----------|-----------|
| Async reset | ![](result_images/dff_async_rst.png) | ![](result_images/d_ff_async.png) |
| Async set | ![](result_images/dff_async_set.png) | ![](result_images/d_ff_async_set.png) |
| Sync reset | ![](result_images/dff_syncres.png) | ![](result_images/d_ff_syncres.png) |

### Step 9 – Special optimisations (mult2, mult8)
Multplying by 2 is essentially performing a left shift operation. Yosys helps achieve this simply by rewiring the inputs to the outputs:

```bash
> synth -top mult2
> abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib   # reports 0 cells
> show
```

![mult2](result_images/abc_mul2.png)
![mult8](result_images/abc_mult8.png)

---

## Tools Used

- [iverilog](http://iverilog.icarus.com/) – Verilog simulation
- [GTKWave](http://gtkwave.sourceforge.net/) – Waveform viewer
- [Yosys](https://yosyshq.net/yosys/) – Logic synthesis
- [Sky130 PDK](https://github.com/google/skywater-pdk) – Standard cell library
