
# Acceleration structure (AS)
- For efficiency, put geomteries into acceleration structure set to reduce ray-triangle intersection tests. 
	- Based on the bounding volume hierachies that contains a set of Axis Aligned Bounding Box , we can follow the hierachical tree structure until we reaches out the actual geometries at the leaf node.
- Unlike software AS, the hierachy structure is implemented by hardware and the end usre only can access in two exposed interface - TLAS and BLAS
	![](../../../../Pasted%20image%2020240315231637.png)
	- Top Level AS : 
		- referencing any number of bottom level AS. 
		- contain the object instances.
		- accessed from shader as a descriptor binding or by device address (via `vkGetAccelerationStructureDeviceAddressKHR`)
	- Bottom Level AS : specified in `VkPhysicalDeviceAccelerationStructurePropertiesKHR::maxInstanceCount
	  It stores actual vertex data, allowing us to store multiple positioned models within a single BLAS
	  
Typically, BLAS is an individual 3D model and TLAS corresponds to an entire scene built by positioning the BLASes.

It is trivial that the order of transformation is applied to BLAS first , then go up to TLAS.

# Reference
- [Nvidia Vulkan Raytracing Tutorial](https://nvpro-samples.github.io/vk_raytracing_tutorial_KHR/vkrt_tutorial.md.html)
- [Ray Tracing In Vulkan](https://www.khronos.org/blog/ray-tracing-in-vulkan)