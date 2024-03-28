
1. I used the same command pool to build both of BLAS and TLAS
2. i didn't get any error while building BLAS.
3. but i got error when i submit commands related to building TLAS

After i separated command pool to create new command buffer for each BLAS and TLAS, i resolved error.


---

# Vulkan command buffer spec

- each command buffer manages state independently.

# Vulkan Command Pool spec

- command pools are opaque objects.
- It is externally synchronized , therefore it must not be used in multiple threads.
	- recording commands on any command buffers allocated from the pool
	- operations that allocate, free and reset command buffers 
	- reset the pool itself

# Synchronization issues
- Just because a command was submitted early in a buffer doesn’t mean it has finished all of its instructions before a command submitted later
- Just because a command buffer was submitted to a queue earlier than other command buffers on the same queue, doesn’t mean that all of its commands (or indeed, any of them) are guarantted to be finished before a later command buffer starts
- Command buffers submitted on different queues similarly do not provide any guarantees

# The barriers

In CPU-side, There are also read-write hazard on shared resources across multiple threads. 
To ensure a thread reading the address of shared resource in a well-defined way, we can apply memory order policy (std::memor_order in cpp) so that we can make a resource invisible while it is begin written by a certain thread.

However, this is more complicated in GPU.
While CPU is deeply pipelined and programmer doesn't think about the different pipeline stages , memory barriers on the GPU is very expensive.

GPU has many different types of memory writes.
For example,
- a fragment shader write to a framebuffer
- a command transition an image buffer from one memory layout to another.
- a vertex shader writes out transform data to a storage buffer.

Hence, we need to be a lot more explicit about barriers in GPU.
In vulkan , a barrier is composed of two separate steps.
1. src / dest access mask : defines what memory must be visible to where
2. src / dest pipeline stage mask : the stage applied to destination stage and src stage must have been published at the point.

The barrier's rule applies to *all commands* submitted prior to the same command buffer and *all commands* submitted in a different buffer ealier on the same queue.

# References

- Vulkan synchronizatoin primer
	- [Part 1](https://www.jeremyong.com/vulkan/graphics/rendering/2018/11/22/vulkan-synchronization-primer/)
	- 