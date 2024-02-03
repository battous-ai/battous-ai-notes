# Chapter 1 Introduction
## 1.1 The Landscape of Computation Accelerators 
- improve performance → but the clock frequencies improves much more slowly
	- find more efficient hardware architectures
		- vector hardwares → 10x gain in efficiency by eliminating overheads of instruction processing
		- complex operations → by minimizing data movement
- yet flexible?
		- [[Turing Complete]]
## 1.2 GPU Hardware Basics
- Division of labor between CPU and GPU 
	- the beginning and ending of the computation require access to I/O devices
		- More API provides I/O services directly on the GPU
			- hide the complexity of managin communication between the CPU and GPU
			- but all assume the existence of a nearby CPU
- ![[image-20240128102815047.png|600]]
	- left (e.g. nvidia GPU)
		- discrete GPU setup
		- separate DRAM memory spaces for the CPU and GPU
			- CPU DRAM
				- low latency access
			- GPU DRAM
				- high throughput
	- right (e.g. AMD’s APU or mobile GPU)
		- integrated GPU
		- single DRAM memory
			- low power
		- private caches
			- ==[[cache-coherence problem]]==
- ==[[unified memory]]==
- GPU cores
	- _streaming multiporcessors_
		- the threads executing on a single core
			- communicate through a scratschpad memory
			- synchronize using fast barrier operations
		- first-level instruction
		- data caches
		- latency hiding to access memory ==by large numbers of threads==?
- high computation throughput
	- balance computational throughput with high memory bandwidth
		- → parallelism in the memory system
			- → include multiple memory channels in GPU
- improved performance on GPU
	- more area to arithmetic logic units
	- less area to control logic
- ![[image-20240128113032785.png|800]]
	- assume a simple cache model
		- no shared data between threads
		- infinite off-chip memory bandwidth
		- MC Region (CPU)
			- a large cache is shared among a small number of threads
				- threads &uarr;  performance &uarr;
		- Valley
			- cache cannot hold the entire working set → performance &darr;
		- MT Region (GPU)
			- multithreading hide long off-chip latency (tolerate frequent cache miss)
				- threads &uarr;  performance &uarr;
##  1.4 Book Outline
- Chapter 2: programming model
- Chapter 3: architecture of GPU core
- Chapter 4: memory system
- Chapter 5: additional research
# Chapter 2 Progamming Model
- exploit SIMD haredware
	- CUDA: MIMD-like programming model
	- at runtime, GPU hardware executes groups of scalar threads (called **warps**) in [[lockstep]] on SIMD hardware
		- the execution model is SIMT
## 2.1 Execution Model
- threads in a compute kernel ares organized into a hierarchg composed of a *grid of thread blocks* consisting of *warps*
- execute groups of thread together in lock-step → to improve efficiency
	- 32 threads → warp
	- warps → thread block or cooperative thread array (CTA)
- compiler and hardware enable the programmer to remain oblivious to the lock-step nature of thread execution in a warp
	- each thread appears to be independent
- threads within a CTA can communicate with each other via a per compute core scratchpad memory (**shared memory**)
	- each  streaming multiprocessor contains a single shared memory
		- the memory space is divided up among all CTAs running on that SM
		- as a software controlled cache
- threads *within* a CTA → synchronize using hardware-supported barrier instructions
- thread *in different* CTAs → synchronize through a global address space (accessible to all threads)
	- expensive
## 2.2 GPU Instruction Set Architectures
### 2.2.1 Nvidia GPU Instruction Set Architectures
- nvidia high-level virtual instruction set architecture → Parallel Thread Execution ISA (PTX)
# Chapter 3 The SIMT Core: Instruction and Register Data Flow
- GPU architecture
	- the SIMT cores that implement the computation
	- memory system
- ![[image-20240128175221982.png]]
	- the pipeline consists of 3 scheduling loops
		- instruction fetch loop
			- Fetch
			- I-Cache
			- Decode
			- I-Buffer
		- instruction issue loop
			- I-Buffer
			- Scoreboard
			- Issue
			- SIMT Stack
		- register access scheduling loop
			- Operand Collector
			- ALU
			- Memory
## 3.1 One-Loop Approximation
 - case of a single scheduler
	 - the unit of scheduling is warp
		 - each cycle the hardware selects one for scheduling
	 - the approximation
		 1. fetch the next instruction to execute for the warp 
			 - by the warp’s program counter 
				 - access a instruction memory
		 2. decode the instruction
		 3. 1) fetch the source operand registers
		 3. 2) determine the SIMT execution mask values (in parallel with fetching source operands from the register file) ?
			 1. ==how to determine? We can only predict?==
		 4. execution proceeds in a single-instruction, multiple-data manner
			- each thread executes on the function unit associated with a ==lane==? provided the SIMT execution mask is set
### 3.1.1 SIMT Execution Masking
- SIMT execution model
	- the abstraction that individual threads execute completely independently
	- achieved via a combination of
		- traditional prediciton
		- a tack of predicate masks (*SIMT stack*)
	- SIMT stack solves 2 issues
		- nested control flow 
		- ==skipping computation entirely while all threads in a warp avoid a control flow path==
- ==how does GPU hardware enable thread within a warp to follow different paths through the code while employing a SIMD datapath that allows only one instruction to execute per cycle?== (a.k.a branch divergence handling mechanism)
	- to serialize execution of threads following different paths within a given warp
	- to achieve this serialization of divergent code paths
		- stack with three entries
			- a reconvergence program counter (RPC)
				- where threads that diverge can be forced to continue executing in lock-step
			- the address of the next instruction to execute (Next PC)
			- an active mask
		- order of the stack entries following a divergent branch
			- entry with the most active threads first
			- then the entry with fewer active threads
### 3.1.2 SIMT Deadlock and Stackless SIMT Architectures
- stack-based implementation of SIMT can lead to a deadlock condition
	- ![[image-20240130105258105.png]]
	- to reach the reconvergence point, all divergent threads must complete computation
		- but the other threads are still waiting for the mutex to release
			- which can only be completed when the reconvergence point is reached
				- hence deadlock
- nvidia new thread divergence management approach since Volta
	- called Independent Thread Scheduling
	- stackless
	- the key idea is to replace the stack with per warp convergence barriers
	- ![[image-20240130120132208.png]]
	- stored in the register and used by hardware warp scheduler
	- *Barrier Participation Mask* 
		- track which threads within a given warp participate in a given convergence barrier
		- may be more than one barrier participation mask for a given warp
		- threads tracked by a given barrier participation mask will wait for each other to reach a common point in the program following a divergent branch and thereby reconverge together
	- *Barrier State*
		- tracks which threads have arrived at a given convergence barrier <u>sdfsdf</u>
	- *Thread State*
		- tracks (the threads in a given warp)
			- is ready to execute
			- blocked at a convergence barrier 
				- if so, which one
			- has yielded
