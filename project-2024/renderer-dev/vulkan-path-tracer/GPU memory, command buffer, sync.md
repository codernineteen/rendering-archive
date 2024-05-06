
# 3. GPU memory

- initialize nvvk memory allocator
- allocate `VkBuffer` for color data on GPU
- map `VkBuffer` data back to CPU

Bandwidth between RAM and VRAM is much slower than bandwidth between RAM and CPU or VRAM and GPU.
![](../../../images/Pasted%20image%2020240502101741.png)

To make transferring between RAM and VRAM faster, 
1. compress data in CPU, transfer the data and decompress it in GPU.
2. minimize the number of transferring itself.

-> Vulkan takes a 'command buffer' approach to improve this problem by recording GPU commands in CPU memory and submit then at once.


## 3.1 memory allocation in Vulkan
- a bit lower level than we're used to
	- For example, malloc in c/c++ is more complex than it appears to end-user
	- we can think of memory allocation in Vulkan as writing GPU version of malloc ourselves
- nvvk resource allocator hold several memory allocators to abstract the steps of memory allocation (create target , allocate memory and then bind)
	1. Dedicated memory allocator - single memory allocation per resource
	2. DMA memory allocator - Nvvk implementation
	3. VMA memory allocator - AMD vulkan memory allocator


## 3.2 Vulkan Buffers (for rendered image)
- similar to a pointer to an array of C++
- need to specify buffer usage.
- In ray tracing extension, we use a buffer to render an image with 'storage buffer' usage flag.
	storage buffer is simply a buffer that **GPU can read and write it** and **can be used as the destination of transfer operation.**
	For storage buffer, we require `host visible` and `host coherent` charateristics
		It means that we still can read and write it in CPU even though the buffer exists in GPU. 

Here is the simple buffer creation with nvvk framework
```cpp
// create a storage buffer
VkDeviceSize buffer_size = render_height * render_width * 3 * sizeof(float);
auto buffer_create_info = nvvk::make<VkBufferCreateInfo>();
buffer_create_info.size = buffer_size;
buffer_create_info.usage = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT | VK_BUFFER_USAGE_TRANSFER_DST_BIT;

nvvk::Buffer storage_buffer = allocator.createBuffer(
	buffer_create_info,
	VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT | VK_MEMORY_PROPERTY_HOST_CACHED_BIT // memory properties
);
```

Actually, Many vulkan applications use `VkImage` to store images such as textures and render frames. However `VkImage` is more complex struct than `VkBuffer`. That's why the storage buffer is used for rendering frame in this tutorial.

## 3.3 Retrieving data from GPU to CPU

```cpp
// retrieve data from GPU to CPU
void* data = allocator.map(storage_buffer);         // map memory to CPU
float* float_data = reinterpret_cast<float*>(data); // cast void data type to float pointer type
printf("%f, %f, %f", float_data[0], float_data[1], float_data[2]);
allocator.unmap(storage_buffer);                    // unmap memory

```


---

# 4. commands

## 4.1 comman buffers & pools

Type of commands :
- fill a buffer with values
- build acceleration structures
- copy an image
- bind pipeline
- etc

Submission order **is not related to** finishing order of each commands. (because of thousands of GPU parallel cores).

If we want to make sure some of tasks complete in the desirable order as we expected, we must apply proper synchronization methods provided by Vulkan 

Available synchronization data structures
- semaphore (between tasks in general)
- fence (between cpu and gpu in general)
- pipeline barriers (for correlated stages)
- events (To achieven asynchronous)

# 4.2  synchronization - pipeline barrier

Should remember two things :
1. two commands start in order, but it doensn't mean the first command will always be finished before second command
2. a command to write memory doesn't mean that subsequent commands will be able to read those written value by the command.
	For this case, we should make it visible before reading it.

Pipeline barrier synchronize memory so that destination pipeline stages can wait until the source stages complete tasks.


### stage mask
A pipeline stage is represented by a `VkPipelineStageFlagBits` bitmask.
Stages in `dstStageMask` will wait for and be able to read the writes of stages in `srcStageMask`.
There are two exceptions :
- `TOP_OF_PIPE`
- `BOTTOM_OF_PIPE`
These are imaginary pipeline stages from graphics pipeline.

### access flags
A single access flag bitmask represents a way that memory can be accessed.
ex) `VK_ACCESS_TRANFSER_WRITE_BIT`, `VK_ACCESS_HOST_READ_BIT`.

Multiple access flags can be stacked by using `or` operation.
It is possible to infer 'access flag' from pipeline stage.
For example, `VK_PIPELINE_STAGE_TRANSFER_BIT` <-> `VK_ACCESS_TRANSFER_WRITE_BIT`

`nvvk::makeAccessMaskPipelineStageFlags` helps us to configure this.

### difference between stage mask and access flags

There are two different dependency context.
- execution dependency - related to pipeline stages
- memory access dependency - related to both of pipeline stages and access flag.

### What pipeline barrier doesn't synchronize

- CPU exectution while running commands in GPU.
	For this, we need to call `vkQueueWaitIdle` or use `VkFence`

### Types of Barriers
- memory barrier
	Create a memory dependency between access scopes, applied to all of memory.
- buffer memory barrier
	memory barrier feature + tranfserring ownership of a buffer between queues.
- image memory barrier
	Most complex one, memory dependency between access scopes and subresource range of an image.
	Perform image layout transition between different image layouts.



# References.
- [vk mini path tracer tutorial](https://nvpro-samples.github.io/vk_mini_path_tracer/index.html#deviceextensionsandvulkanobjects/requestingdeviceextensions)