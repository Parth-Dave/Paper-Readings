# Reconfigurable Architecture for Embedded Processing

Embedded systems require a processing unit, as well as many other diverse peripheral devices, for instance - sensors, storage devices, communication modules, etc; and thus, a lot of interfacing is required. These additional modules contribute to the area and power overhead of the overall system.

These are implemented in two ways : hard-core processors and soft-core processors. Only the latter provides reconfigurability, which is preferred for embedded systems.

However, this additional flexibility comes at a cost of higher power, worse operating time and a larger die area. This is what the paper aims to bring down.

# Overview

1. [Existing Implementation](https://www.notion.so/Reconfigurable-Architecture-for-Embedded-Processing-352c7e7c9be941498ada9af9bdad79de#a3fa3f626a8144e7ad5f18e5587bc4e3)
2. [Analysis of a RISC-V Processor](https://www.notion.so/Reconfigurable-Architecture-for-Embedded-Processing-352c7e7c9be941498ada9af9bdad79de#2f4f4520883840dc8325a11d781cb467)
3. [Design Methodology](https://www.notion.so/Reconfigurable-Architecture-for-Embedded-Processing-352c7e7c9be941498ada9af9bdad79de#10be672046d949e6ada91bc362a3670d)
    1. Identification
    2. Grouping
    3. Implementation
4. [Results](https://www.notion.so/Reconfigurable-Architecture-for-Embedded-Processing-352c7e7c9be941498ada9af9bdad79de#5c1634b4a36840d3bbc0903b939b9dd9)

# Existing Implementations

1. Power Gated Sub-Modules : the processor is implemented with all the blocks required to carry out instructions. However, their power supply is gated with control signals.

    ^ Area Inefficient, Non Reconfigurable

2. Coarse Grained Reconfigurable Modules : Reconfigurable modules are used instead, implementing entire co-processors altogether.

    ^ Area Inefficient, Require a compiler overhead

3. Fine Grained Reconfigurable Modules : Reconfigurable modules are worked into the existing pipeline. Compiler may or may not optimise code.

    ^ Area is better, but power consumption goes up.

    ^ Moreover, modifications would need to be made to the microarchitecture as far as the bus lines and data-path are considered.

# Analysis of a RISC-V Processor

### 1. Area and Frequency of Utilisation

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled.png)

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%201.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%201.png)

**Observation** : Lots of sub-modules consume a lot of area, but have a low utilisation percentage.

⇒ To save area, club these low utilised modules into reconfigurable blocks.

---

### 2. Access Patterns

Consider 2 (sporadically used) components A and B, with equal utilisation percentages (say 50-50). They could have the following two sample patterns of utilisation :

- {A, A, A, A, A, B, B, B, B, B, A, A, A, A, A, B, B, B, B, B, ...}
- {A, B, A, B, A, B, A, B, A, B, A, B, A, B, A, B, A, B, A, B, ...}

Clearly in the second case, even though the utilisation percentage is the same, the latency of instructions will take a massive hit if a reconfigurable module was used to implement it.

**Observation** : The frequency of reconfiguration would end up causing severe delays in the instruction flow of this program.

⇒ Have a metric to judge which modules should be clubbed together, or which modules should be left alone.

The metric described in the paper is called the Switching Rate (SR). It is equal to the number of reconfigurations divided by the total number of instructions.
*Case 1 : 4 / 20 = 0.2
Case 2 : 10 / 20 = 0.5*

# Design Methodology

[Done by taking the RISC-V processor into consideration]

Instructions in the ISA would determine microarchitecture of the processor. How that is implemented can be thought of across two scales :

1. How many instructions are clubbed into a block?
2. Are these blocks implemented in hard logic or reconfigurable through LUTs?

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%202.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%202.png)

---

### 1. Identification of Modules

At first, the modules that would be involved in the reconfigurable architecture can be identified. The characteristics that would lead them to be classified as such would be a high area and a low utilisation percentage ⇒ FP modules, MUL/DIV modules, **not** ALU, and so on...

---

### 2. Grouping of Modules

Secondly, these identified "Candidate Units" (CUs) need to be grouped into "Reconfigurable Units" (RUs). In order to keep this optimal, an "Efficiency Goal" is defined.

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%203.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%203.png)

T_exec would consist of two terms : 
    1. Time required to execute the instruction
    2. Time required for reconfiguration

Out of the two, the first can be more or less neglected.

Thus, the total efficiency goal for RUs can be given as :

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%204.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%204.png)

The clubbing of units can be done differently depending upon what the application is. This would be derived from the access patterns observed in the benchmark.

E.g. if the Access Pattern looks like {A, B, A, C, A, B, C, B}, then :

If A and B are clubbed, the switching rate would be 3 / 8

If B and C are clubbed, the switching rate would be 4 / 8

If A and C are clubbed, the switching rate would be 3 / 8

Thus, A would be clubbed with either B or C, depending upon either the similarity of the Boolean Function A with those of B or C, or which out of B and C require an area similar to that of A.

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%205.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%205.png)

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%206.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%206.png)

---

### 3. Implementation of Modules

Third, how these reconfigurable units are implemented is discussed. As mentioned before, RUs can be implemented through a combination of Lookup Tables (LUTs) and Configurable Hard Logics (CHLs).

It's been shown that the use 4-LUTs in FPGAs leads to the least area among other such LUTs. Despite using these, a large space corresponding to their structure remains underutilised. Even if this is mitigated, small sets of specific functions may repeat across different functions; i.e. for n 4-input functions, (2^4)*n SRAM units won't always be required. The high number of SRAM cells has a significant impact on the area and power consumption of the entire system.

Shown below is the implementation of the combinational boolean function

Y = A'B'CD + (A'BC'D + A'BCD') + ABC'D' + (AB'C'D + AB'CD')

= A'B'CD + ABC'D' + (A^B).(C^D)

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%207.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%207.png)

For optimisation at this level, another modification is suggested : to use an area-power optimised solution; i.e. instead of pure LUTs, use a CHL-LUT based RU.

To compare CHLs with LUTs, it is first seen that in a 4-LUT, with the presence of 16 SRAM cells make it possible to implement any arbitrary 4 input function out of the total (2 ^ 16) unique boolean functions that exist. Since the use of a few select family of functions (say A + B + C + D) may be far more abundant than other specific ones (say A'B' + C' + D), a Hard Logic is implemented which can emulate these large families of functions. CHLs take advantage of NPN equivalence of boolean functions [[Further explanation](https://www.notion.so/Reconfigurable-Architecture-for-Embedded-Processing-352c7e7c9be941498ada9af9bdad79de#e083a14823e4449881fb70c6ca181e04)].

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%208.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%208.png)

As it turns out, two different kinds of CHLs can cover over 60% of all binary functions through clever design, and these can be incorporated in the basic building blocks that would be used to implement all reconfigurable modules.

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%209.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%209.png)

---

# Results

The results were found by carrying out a top level simulation of the firmware, right down to the schematic levels. Thus, small caveats relating to transistor sizing, timing analysis and clustering / placement was done. Alongside these calculations, the HDL programming for the Configurable Logic Blocks was done. Lastly, the sample test-bench is analysed to determine the utilisation percentage and grouping of modules. And thus, the time and power consumed by running a high level language on this hardware can be estimated.

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%2010.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%2010.png)

### Reduction in Area

The number of LUTs is reduced by 26.8%. Along with the reduction in configuration bits, logic cells and routing resources, the total reduction in area in the proposed architecture is 30.9%.

### Reduction in Critical Path Delay

The logic delay for individual modules is not affected, because the logic design and mapping is the same (say for the FP-Double unit). Secondly, the routing delay goes down as the channel width for Reconfigurable Units is considerably lower in comparison to the Reconfigurable Module architecture (LUT based). This reduction amounts to 9.2%.

### Reduction in Application Execution Time

The execution time for each unit was multiplied by the number of times its corresponding instructions were called. To account for reconfiguration time, the write latency was multiplied with the number of configuration bits for each module (divided by 128, i.e. the width of the data bus that interfaces the SRAM with the RUs). Total total improvement amounts to 6.2%.

### Power and Energy Consumption

After accounting for all the modules (utilised as well as dormant), there's a 32.5% reduction in power consumption. Static power is reduced for both logic cells and routing resources. However, conventional ASIC implementation would always be more efficient than a reconfigurable one.

Another important set of improvements over conventional solutions as mentioned earlier have a lot to do with avoiding power gating. The following issues come up with the technology :

1. Inrush current : modules turning on abruptly can cause instability of register contents and cause functional errors because of dynamic voltage drops.
2. Moreover, wake-up energy (inrush power) contributes to about 20% of the static power consumption in FPGAs.
3. Power gating requires the compiler to predict the idle and utilisation time of resources, in order to make decisions as to when power should be turned off / on.

### Additionally

The use of CHL-LUT based RUs also reduces most parameters, as summarised in the table below.

![Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%2011.png](Reconfigurable%20Architecture%20for%20Embedded%20Processin%20746c3ed5b5b5405e8a687edc4e8cd66a/Untitled%2011.png)

---

The basis of CHLs lies deeply within boolean algebra and is specifically used to exploit the equivalence classes of functions. The idea of a CHL is that a few gates are connected in a certain layout or pecking order. A few select internal and external signals are gated to provide the ability to be negated. Moreover, between the n (n=4) inputs and these gates, a re-ordering or permuting gate is placed. The result of this is an overall reconfigurable gate which can implement a lot of boolean functions with this flexibility.