# Trace cache

[Trace Cache paper](http://www.eecs.harvard.edu/cs146-246/micro.trace-cache.pdf)

# Problem

- Fetch Bandwidth:- With increase in issue width of super-scalar architectures, fetch bandwidth is becoming a bottleneck. More than 1 control flow executed each cycle
- Non Contiguous Instruction Alignment :-Conventional caches cannot fetch multiple blocks each cycle as instruction issue order is not always contiguous memory locations

# Existing Mechanisms

- Branch Address Cache :- Extension to BTB, Hits in BTB gives single address for next block. Similarly tree like structure in cache combined with multiple branch predictions give starting address for multiple blocks. These are fed into highly interleaved cache (multiple address input and data output)
- Collapsing buffer :- Detects forward branches in cache lines and eliminates between instructions. Good for short forward branches
- All these techniques have issue like producing all block address before fetch, simultaneous access to multiple lines in cache , aligning multiple blocks dynamically before decoder

# Idea presented by paper

- Increase number of predictions every cycle (Branch Throughput)
- Add an extra cache which stores already fetched blocks from non contiguous locations in each line (Non contiguous access)
- Negligible impact to conventional cache fetch latency and prediction penalty (Fetch Unit Latency)
- Trace cache stores dynamic sequences instead of trying to find and fetch them hence it doesn't add to critical path because in case of miss conventional cache is used.

# Details

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled.png)

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%201.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%201.png)

## Core fetch unit

- Interleaved cache to access two lines at same time for sequence to span cache line boundaries
- Full cache line upto first taken branch
- All BTB lines accessed in parallel to check for branches in instructions being fetched , connected with RAS and jump check
- High accuracy correlated branch predictor

 

## Trace Cache

- Max N instructions (defined by issue width) or m branches(by max predictions per cycle) per line
- valid bit
- tag :- starting address of trace
- branch flags :- m-1 bits to track if branch taken or not taken in trace
- branch mask:- number of branches in line to check with predictions and if last instruction in trace is branch
- trace fallthrough :- next address if last branch not taken
- trace target :- next address if last branch taken
- trace cache accessed in parallel with core fetch unit and used with 2:1 MUX with trace hit as selector
- if miss then normal fetching , instruction blocks merged added to line fill buffer
- approx 4.7kb for n=16,m=3 and 64 lines

# Further Improvements

- increase associativity of trace cache
- Having multiple paths by path associativity
- partial matches till correct predictions
- Indexing by combining prediction bits with start address
- improved line fill
- small buffer before committing in cache to check for reuse or victim cache

# Results

- n= 16, m= 3 , 2048 instruction buffer
- all units other than fetch are ideal
- MIPS ISA
- unlimited register renaming, load and store stall only with true data dependency, data cache always hits, dispatch latency 1 cycle, operation latencies of MIPS R10000

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%202.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%202.png)

- SEQ 1 :- 1 block per fetch
- SEQ 3 :- 3 contiguous blocks per fetch
- CB: collapsing buffer
- BAC: Branch address cache
- TC: Trace Cache

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%203.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%203.png)

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%204.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%204.png)

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%205.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%205.png)

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%206.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%206.png)

![Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%207.png](Trace%20cache%20a1fa1112c69e422c8f2d95fd443f854a/Untitled%207.png)
