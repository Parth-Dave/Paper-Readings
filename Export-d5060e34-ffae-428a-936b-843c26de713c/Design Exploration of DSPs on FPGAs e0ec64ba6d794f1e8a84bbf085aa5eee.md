# Design Exploration of DSPs on FPGAs

The paper aims to minimise area for an implementation of a DSP on an FPGA, given a minimum throughput constraint.
The premise requires selection of modules : 
    - May be slow but more area efficient
    - May be faster and larger modules but can be time-multiplexed

Manually making these decisions can be tedious. This paper therefore introduces a Design Exploration Methodology (using two different algorithms), considering pipeline scheduling and resource sharing.

# Overview

1. Context : [High Level Synthesis](https://www.notion.so/Design-Exploration-of-DSPs-on-FPGAs-d2d4e46f52aa43e5abfcdb631236d8a7#dcea43f13c0a4c559c25b50de2b6f3e6)
2. [Pipeline Scheduling](https://www.notion.so/Design-Exploration-of-DSPs-on-FPGAs-d2d4e46f52aa43e5abfcdb631236d8a7#ba1d39e07c364a97aa779ec297c7252c)
3. [Resource Sharing](https://www.notion.so/Design-Exploration-of-DSPs-on-FPGAs-d2d4e46f52aa43e5abfcdb631236d8a7#c7919744a5414aaea218781de9c447e1)
4. [Module Selection](https://www.notion.so/Design-Exploration-of-DSPs-on-FPGAs-d2d4e46f52aa43e5abfcdb631236d8a7#908021523f18436f96c1076045ffc5f4)
5. [Traditional Approaches](https://www.notion.so/Design-Exploration-of-DSPs-on-FPGAs-d2d4e46f52aa43e5abfcdb631236d8a7#8456d96cc7a648ba9dff59576f4eb965)
6. [ASAP Branch and Bound Exploration](https://www.notion.so/Design-Exploration-of-DSPs-on-FPGAs-d2d4e46f52aa43e5abfcdb631236d8a7#58cc0c3710c64f50ae254a32f8a6cb73)
7. Iterative Modulo Exploration
8. Results

Pipelining is important in the case of FPGAs because all the configurable blocks and routing resources lead to a high propagation delay. Moreover, even though (pipeline) registers may be expensive, FPGAs would have ample general purpose registers to work with.

Pipelining also helps a circuit work faster (better clock rate). This may not be useful for a DSP with a maximum given sampling rate. Thus, the goal of DSP design on FPGAs usually is driven by the area. Pipelining can help here too.

# High Level Synthesis

If a certain computation needs to be carried out repeatedly on a dataset, customised hardware can speed things up. For CPUs, since a large instruction set needs to be supported, complex calculations should be programmed in software.

However, specialised hardware modules such as DSPs need to perform functions such as complex multiplication, repeated addition and multiplication over an array, FFT, and more such functions.

If this calculation to begin with is specified in MATLAB, or C, a data-flow graph is made at first. This data flow graph can then be implemented through an HDL, before the final circuit is synthesised. Without any constraints, multiple implementations are possible.

E.g. - a[n]+jb[n] = (a[1] + jb[1]) * (a[2] + jb[2])

**a[n]** = a[1]*a[2] - b[1]*b[2],     **b[n]** = 2 * (a[1]*b[2] + a[2]*b[1])

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled.png)

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Complex_Mult_ASAP.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Complex_Mult_ASAP.png)

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Complex_Mult_MulitCycle.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Complex_Mult_MulitCycle.png)

# Pipeline Scheduling

The incorporation of a pipeline usually allows two iterations or two instructions to overlap with respect to time. This leads to an increase in throughput since the maximum system clock frequency increases.

However, with multi-cycle implementations of a single operation, resource consumption can be reduced, and pipelining might become tougher.

Different time and resource constraints can be analysed more systematically with the introduction of certain parameters :

### Top Level Parameters :

- Data Initiation Interval (δ) - How many clock cycles separate two instructions, data points, or iterations.

### Module Parameters :

- Module Latency (λ_m) - Combinational delay (in clock cycles).
- Module Data Initiation Interval (δ_m) - Clock cycles separating two pieces of data into the module.
- Module Area (A_m)
- Max Frequency (f_m) - Maximum frequency that the module can operate at.
- "Shareable-ity?" (Share_max) - Loose idea of how many operations can be shared on a module. This isn't dependent on just a module; it depends on δ.

### Constraints :

- δ ≤ floor(min(f_m) / Throughput)
- δ ≥ max(δ_c) - Maximum data initiation interval of recursive cycles in the dataflow description.

    ![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%201.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%201.png)

    ![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%202.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%202.png)

- Share_max ≤ floor(δ / δ_m)

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%203.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%203.png)

Fully Pipelined Circuit Modules - δ_m = 1, λ_m = 1
Partially Pipelined Circuit Modules - 1 < δ_m < λ_m
Non Pipelined Modules - δ_m = λ_m

### Design Exploration with these Parameters

For a specialised circuit such as a DSP, a fixed sampling rate implies a constant throughput that needs to be met by the circuit in question. As a result, different circuits can be synthesised at different operating frequencies. For e.g. if the throughput is 50M samples / s, the design space would consist of the following parameter sets : (50MHz, δ=1), (100MHz, δ=2), (150MHz, δ=3)...

Thus, this paper provides a method to perform design exploration over a wide range of (f, δ) pairs.

# Resource Sharing

Resource sharing will require a certain amount of modifications to the original datapath. With modules being used over multiple cycles, three additional functions (costs) would be required :

1. Multiplexing Logic
2. Routing Resources
3. Registers for Storage (?)

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%204.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%204.png)

# Module Selection

Each operator in the dependence graph might have multiple implementations. The goal while considering these implementations would be to choose the module with the most desirable characteristics, i.e. the least area, or the best timing in some cases. For this approach, there must be a library containing all these implementations, along with a pairing of the operations each of them can support. With this in place, the problem can be put into perspective.

The entire "design space" will not only contain each possible combination of every compatible module (m_i) for each node (n_i), but also designs which will have a different dataflow chart from the original one.

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%205.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%205.png)

Each of these different combinations, or designs will have different characteristics. Area is one characteristic that has been the focus of this paper's optimisation. It defines cost as the number of slices taken on an FPGA to implement that circuit, provided it meets the throughput constraint.

# Traditional Approaches

Conventionally, only one out of module selection and resource sharing is performed before a circuit is implemented for an HLS problem. Even if both are done, the circuit that is eventually synthesised might not be as efficient as the optimal solution.

If only module selection is done, each operator is allotted a module, and this assignment is done for all (f, δ) pairs. Since no resource sharing is done, the cost only changes according to the individual costs of all modules that support an operation. However, there wouldn't be any scope for pipelining.

^ This is computationally inexpensive.

If only resource sharing is done, a certain pairing (or combining) of resources wouldn't be valid throughout the complete (f, δ) set. As a result, multiple combinations will have to be checked, and that alone will cause the computational overhead to limit design exploration. Moreover, for each pairing, there wouldn't be a fixed cost of multiplexing and routing. Thus, it'll need to be determined separately.

^ Computationally expensive, but arguably provides better area optimisation.

### Combining MS and RS

If both of these techniques are done during design exploration, the order in which they're done causes a massive change in computational overhead too.

1. If RS is done before MS, the design space would be made smaller because not all modules would be shareable. However, all constraints would be put right at the start, and an optimal solution would have a small chance of staying within the design space.
2. If MS is done before RS, there won't be any certainty of resource sharing, simply because the use of really efficient modules would limit their ability to be shared among operations.

Essentially, if MS and RS are done together, and simultaneously, the resulting circuits could be highly optimal.

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%206.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%206.png)

# ASAP Branch-and-Bound Exploration

### ASAP Scheduling

^ an operation is scheduled right as soon as its input data is present; a new module will be used if there are no available modules.

The algo on the right iterates through operations.

1. For each op, it checks if there are already any modules which it can be combined with. If that is the case, resource sharing is considered, and rescheduling is done with the new dependencies.
2. If no such shareable module exists, a new one is selected. The modules that don't satisfy the circuit parameters are eliminated. This is where the design space reduction takes place.

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%207.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%207.png)

This ^ algorithm does give out better solutions, but it's still computationally expensive. On average, its complexity can be estimated to be of the order O(M^N), where M → number of modules for each node (operation) and N → number of operations in total.

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%208.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%208.png)

# Iterative Module Exploration

The outer loop iterates bw δ_min and δ_max.

1. For each δ, it calculates an operating frequency.
2. A pre-determined number of iterations is fixed.
3. For each sub-iteration, random scheduling is carried out of the total module space*. And an approximate 

![Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%209.png](Design%20Exploration%20of%20DSPs%20on%20FPGAs%20e0ec64ba6d794f1e8a84bbf085aa5eee/Untitled%209.png)