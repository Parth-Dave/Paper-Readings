# Pipelined Dataflow Processor

# Control Flow Computing (Von Neumann)

Conventional processors use this type of computing and Instruction Set Architectures like RISC-V or x86 and are based on principles of control or program flow computing. This works mainly using the instruction data as framework. This type of architecture uses a control unit, an ALU, a memory unit, registers etc.

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled.png)

# Data Flow Computing

In this type of computation, the instruction execution is on the basis of data availability. This means that the execution can be asynchronous and systems can be designed such that executing can be done as soon as the required data is available. This shaves off any stand-by time that a program-flow computation would entail. This allows more parallelism and simultaneous asynchronous processing to occur.

- Used for specialized hardware in DSP, graphics processing, network routing etc.
- Due to the scope of parallelism, it has good potential in HPC.

## Features

- Intermediate or final results can be passed as tokens in a more transient form.
- Shared memory in the form of variables is not required.
- Programming is constrained by the data dependency.
- The machine language in this case is the data-flow graph.
- Computation occurs at the "nodes" in the data-flow graph.
- For any timing requirements control tokens are generated, and used to trigger executions or channel data-flow.
- A single memory (instead of separate program and data memory) is used so the load/store doesn't need to happen.

## Working

- The program is written in order of execution after considering data dependencies.
- The compiler creates unique tags for each dependency and assigns tokens to enable the execution at various nodes.
- Once all required data operands have arrived, the execution happens at the nodes, and this process is called firing of the node.
- The output, in the form of a result token, is then passed to the subsequent node and so on.
- Once the result token is sent to the input arc of the subsequent node, the output arc is emptied.
- Dataflow can be steered using the likes of conditional nodes for more complicated algorithms.

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%201.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%201.png)

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%202.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%202.png)

## Data Flow Nodes

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%203.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%203.png)

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%204.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%204.png)

## Static vs Dynamic Dataflow

In static dataflow only 1 instance of a node can be executed, and there is only 1 enabling token for this node. So, in case there are 'n' instances of the same node, not only must the storage be provided for 'n' operators but also these will have to run at different data points.

In dynamic dataflow, multiple instances of a node can be executed at the same node using 'n' frames to take the input arguments and give outputs. However, since there are multiple frames of the same node, the 'n' enabling tokens for this node need to be unique. To allow for this the tokens must now be tagged such that each can be uniquely identified. This way, a frame can be allocated to each instance of a node, for example in a loop iteration.

## Advantages

1. Execution occurs as soon as data arrives, instead of waiting for clock edges.
2. Tokens are used instead of variables, so the memory is temporary.
3. Extensive scope for parallel computing with minimal space and power consumption.
4. The load/store operations need not happen consecutively for instructions.

## Disadvantages

1. Much harder to code.
2. It's asynchronous computation so separate synchronizing signals must be passed instead.
3. Data dependencies must be analysed globally before creating the source program.
4. Memory locality cannot be exploited, due to single memory.

## Implementation of argument-fetch dataflow architecture

Dataflow architecture works on the basis of having several dataflow cores (or execution units) with each being referred to as an actor that's defined to work in accordance with the standard dataflow rules.

An argument-flow dataflow architecture usually involves an actor with 2 input arcs and all corresponding output arcs. The output arcs interconnecting subsequent nodes serve as channels to sending the result token itself, signalling that an input is available, and receiving ACK from successor so that the result token can be removed.

By evaluating several hardware schemes, the paper finds it to be further optimal if the roles of signalling and passing of result token are separated. An argument-fetch dataflow architecture involves the operands being fetched by each target instruction from the data memory instead of having the predecessor actor store the result in the operand field of each target instruction.

This design is split into two main components:

Dataflow instruction scheduling unit (DISU) - for the dataflow signal graph.

Pipeline instruction processing unit (PIPU) - enables instructions in pipeline such that there's no data conflict.

The PIPU has the set of instructions, P, but does not know the sequencing of them.

The DISU has the signal flow, S, where each arc specifies how the executions in subsequent actors are connected.

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%205.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%205.png)

In the above diagram, it can be seen that each node in the SFG contains a reference to the set of instructions in the PIPU. As evident, there is no program counter or PC+4 happening in this architecture. The sequencing is decided solely on the basis of the SFG given in DISU. 

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%206.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%206.png)

The "fire" signals represent sending of instruction addresses to be executed. For e.g. 'n1' as in case of the above SFG. Subsequently, 'n2' and 'n3' will also be fired and this will probably be before the "done" signal to acknowledge 'n1' is received. This way the instruction pipeline can be filled, and as ACK is received further "done" signals can be sent to keep the process going till the entire SFG is traversed. This entire process will be maximally optimal if the rate of firing becomes equal to the avg. rate of execution in PIPU.

## PIPU

This is far more optimal than the execution pipeline of a RISC architecture, primarily because the RISC pipeline is limited to only a few stages because of the sequential programming. In dataflow architecture, several independent blocks of the code can be used to fill the pipeline simply because of the parallelism involved. Now each of these 4 stages can be split into several stages, based on the respective timing overheads. This can utilize the fact that a dataflow architecture can support more pipeline stages than RISC architectures, due to a larger pool of instructions. This translates to a reduction in the memory access overhead per cycle of pipeline.

![Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%207.png](Pipelined%20Dataflow%20Processor%206ff93c1f566a440e81a1ff0b48b56c3c/Untitled%207.png)

## DISU

The DISU consists of a signal processing unit (SP) and the enable controller unit (EC) which conduct the primary function of the DISU, which is to issue the firing signals in accordance with the SFG. The arriving done signals are counted and matched with the required amounts of "done" signals required to issue the fire signal for the subsequent node. For e.g. if a particular actor requires the operands p and q, the "fire" signal for that actor will not be issued until the "done" signals are received for the actors with result tokens p and q. There may also be several other tokens that are required before a node can be fired, for e.g. in case of conditional nodes. A count of required signals for each node is kept in the DISU and the fire signals for any node can be issued the moment all the required signals are received. Due to the parallelism, this can be for several distinct parts of the code, and these firing signals may be issued simultaneously as well in order to keep the pipeline operating at full capacity.

## Conclusion

- Pipelined execution unit can be kept running at full capacity if DISU operates in accordance.
- The problem of data dependency is solved by the concept of dataflow itself.
- The pipelined instructions need not be from the same part of the code.
- The argument-fetch implementation is more complex, however, it gives the ability to exploit memory locality.
- The compiler and code must globally analyse all dependencies before generating the dataflow.