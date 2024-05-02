
---
# 1.1 Context
![](../../../images/Pasted%20image%2020240501091831.png)

# 1.2 Instance extensions & layers

- extensions : additional functionalites that GPU can do over core functionalities
- layers : an intermediate between application and GPU driver (ex. Validation layer for debugging)

# 1.3 Physical device and Logical Device
- physical device : represent actual GPU device which is discrete or integrated
- logical device : interface of one of physical devices. Provides device handle
![](../../../images/Pasted%20image%2020240501093100.png)
It is possible to use more than one GPU at the same time in a single application.

# 1.4 Queue
## workflow
1. allocate command buffer from command pool
2. begin recording -> record commands -> end recording
3. submit commands to queue

## queue type
- graphics queue : do graphics operations (ex. ray tracing, rasterization)
- transfer queue : support data transfer operations (ex. clearing and copy data)
	graphics queue can provide transferring data either.

---

# 2. Device extensions & vulkan objects

## 2.1 requesting device extensions for ray tracing

- `VK_KHR_acceleration_structure` - enables acceleration structures to cast rays on geometries quickly
- `VK_KHR_ray_query` - enables **compute pipeline** and **graphics pipeline** to cast rays
- `VK_KHR_ray_tracing_pipeline` - adds **ray tracing pipeline** (shader binding tables, descriptor sets for ray tracing pipeline)
![](../../../images/Pasted%20image%2020240501095749.png)


---

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


# References.
- [vk mini path tracer tutorial](https://nvpro-samples.github.io/vk_mini_path_tracer/index.html#deviceextensionsandvulkanobjects/requestingdeviceextensions)