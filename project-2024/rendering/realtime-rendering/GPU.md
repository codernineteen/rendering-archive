
- data transfer is considered as the main bottleneck in GPU than other operations

# data-parallel architectures

**CPU**
- can have multiple processors (but relatively small set)
- a wide variety of data structures and complex control flow available.
- but mostly run the codes in a serial manner.
- to minimize latency between main memory and secondary memory, CPU has a cache
- CPU avoid stalls by using branch predication, instruction reordering, register renaming, cache prefetching

**GPU**
- has a huge set of shader cores (multiprocessor in GPU)
	- GPU's processor is a stream processor (ordered sets of similar data are processed)
- 'Single Instruction Multiple Data' operations handled.
- can handle data in a massively parallel manner.
- GPU is optimized for *throughput* - defines maximum rate at whice data can be processed
	- However, with less chip area dedicated to cache memory and control logic, latency for shader core is higher than the CPU's in general.
- Transferring texture takes almost hundereds to thousands clock cycles (huge bottleneck)
	- In shader, each fragment has a little storage space for its local registers.