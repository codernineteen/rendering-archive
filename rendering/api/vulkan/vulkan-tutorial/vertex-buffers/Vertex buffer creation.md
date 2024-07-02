Buffers - regions of memory used for storing arbitrary data that **can be read by the graphis card**.
Unlike other vulkan objects, we should handle memory management for buffers on our own.

# Buffer creation

`VkBufferCreateInfo`
- sType
- size : (type size of each elements) x (number of elements)
- usage : the purpose of this buffer creation
- sharingMode : a buffer can be owned by a queue famile OR can be shared between multiple queues.
	- vertex buffer will only be used for graphics card -> set exclusive access
- flags : configure sparse buffer memory

Buffers need to be destroyed explicitly by us.

# Memory requirements
Now it's time to assign memory for the buffer.
1. query memory requirements with `vkGetBufferMemoryRequirements`
	params of `VkMemoryRequirements`:
	- size : required amount of memory in bytes
	- alignment : where the buffer begins in the allocated region of memory (depends on bufferInfo.usage & flags)
	- memoryTypeBits : bit field of the memory types. Bit i is set if and only if the memory type i in the `VkPhysicalDeviceMemoryProperties` structure is supported
	Combine requirements of buffer first and then find the right type of memory to use it.

2. query info about the available types of memory. (`vkGetPhysicalDeviceMemoryProperties`-`VkPhysicalDeviceMemoryProperties`)
	`VkPhysicalDeviceMemoryProperties`
	- memoryTypes : a set of memory properties that can be used with a given memory heap . This array consists of `VkMemoryType` structs
	- memoryHeaps : distinct memory resources like dedicated VRAM and swap space in RAM
	
	`VkMemoryType` - specify the heap and properties of each type of memory

# Memory allocation
After determining the right memory type, allocate the memory by filling `VkMemoryAllocateInfo` struct.

- fill allocation info (sType, allocationSize, memoryTypeIndex)
- call `vkAllocateMemory` by passing a parameter `VkDeviceMemory` to point to the address of allocated space.
- bind buffer to the memory

# Filling the vertex buffer
Mapping the buffer memory into CPU accessible memory with `vkMapMemory`

1. `vkMapMemory`
2. memcpy the vertex data to mapped memory.
3. `vkUnmapMemory`

Because the driver may not immediately copy the data into the `buffer memory`, writing to buffer may not be visible in the mapped memory yet.
To deal with this problem :
- Use a memory heap that is host coherent
or
- call `vkFlushMappedMemoryRanges` after writing to the mapped memory.
	then, call `vkInvalidateMappedMemoryRanges` before reading from the mapped memory.
explicit flush has better performance, but it doesn't matter (mentioned in tutorial)

With above methods, the driveer will be aware of our writes to the buffer, **but it doens't mean that they are actually visible on the GPU**
-> Transfer data to GPU is background work.
-> it is guaranteed to be complete as of the next call to `vkQueueSubmit`

# Binding the vertex buffer
Now what we need to do is binding vertex buffer during rendering operations.

Add binding inside of a function recording commands