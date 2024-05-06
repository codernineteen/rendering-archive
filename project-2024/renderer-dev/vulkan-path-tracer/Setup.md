
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






# References.
- [vk mini path tracer tutorial](https://nvpro-samples.github.io/vk_mini_path_tracer/index.html#deviceextensionsandvulkanobjects/requestingdeviceextensions)