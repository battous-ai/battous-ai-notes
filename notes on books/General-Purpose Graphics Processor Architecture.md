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





