
# Acceleration structure (AS)
- For efficiency, put geomteries into acceleration structure set to reduce ray-triangle intersection tests. 
	- Based on the bounding volume hierachies that contains a set of Axis Aligned Bounding Box , we can follow the hierachical tree structure until we reaches out the actual geometries at the leaf node.
- Unlike software AS, the hierachy structure is implemented by hardware and the end usre only can access in two exposed interface - TLAS and BLAS
	![](../../../../images/Pasted%20image%2020240315231637.png)
	- Top Level AS : 
		- referencing any number of bottom level AS. 
		- contain the object instances.
		- accessed from shader as a descriptor binding or by device address (via `vkGetAccelerationStructureDeviceAddressKHR`)
	- Bottom Level AS : specified in `VkPhysicalDeviceAccelerationStructurePropertiesKHR::maxInstanceCount
	  It stores actual vertex data, allowing us to store multiple positioned models within a single BLAS
	  
Typically, BLAS is an individual 3D model and TLAS corresponds to an entire scene built by positioning the BLASes.

It is trivial that the order of transformation is applied to BLAS first , then go up to TLAS.

# Create acceleration structure
- determine the sizes required for the AS
	- size of AS and the scratch vuffers can be retrieved by `vkGetAccelerationStructureBuildSizesKHR`
	- shape and type of AS is created by describing `VkAccelerationStructureBuildGeometryInfoKHR`.
	- `VkAccelerationStructureBuildRangeInfoKHR` : a reference to the range

# TLAS
- the entry point in the ray tracing scene description.
- stores all the instances
	**Instance**
	1. an instance is represented with `VkAccelerationStructureInstanceKHR`, which stores its transform matrix.
	2. it has its identifier that will be available during shading as `gl_InstanceCustomIndex`
	3. Also, the index of the hit group that represents the shaders that will be invoked upon hitting the object.
	The index and the notion of hit group are tied to the definition of the raytracing pipeline and the Shader binding table.

# Ray tracing descriptor set

we can't know which objects will be hit by ray in advance, so **any shader may be invoked at any time.**
-> use a single descriptor set that contains every resources neccessary to render the scene. (ex. all texture for all the materials.)

+ Acceleration structure holds only position data, so we need to pass index and vertex buffer to look up other vertex attributes.

1. make new descriptor compatible to rasterization pipeline.


# Ray tracing pipeline

- Load and compile shaders into `VkShaderModules` 
- pack the shader modules into an array of `VkPipelineShaderStageCreateInfo`
- Create an array of `VkRayTracingShaderGroupCreateInfoKHR` as an entry point of Shader binding table.
- compile above two arrays and pipeline layout into a **raytracing pipeline** using `vkCreateRayTracingPipelineKHR`
- query shader handles by calling `vkGetRayTracingShaderGroupHandlesKHR`
- allocate a buffer with `VK_BUFFER_USAGE_SHADER_BINDING_TABLE_BIT`

## entry point for ray tracing
- ray generation shader (XXX.rgen)
	-  this shader is called for each pixel.
	- initialize a ray starting point based on camera lens model.
	- invoke traceRayEXT() -> shoot rays into scene.
	- `ratPayloadEXT` or `rayPayloadInEXT` :  the variables hold ray trace payloads.
		-> establish caller - callee relationship, and create a local copy of its declared `rayPayloadEXT` variables.

Additional shader types should be used :
- **miss** shader - executed when there is no hit between geometries and current ray at all.
- **closest** hit shader - called upon hitting the geometric instance closest to the starting point of the ray. Then perform lighting caculations and return the results through the ray payload.


# Reference
- [Nvidia Vulkan Raytracing Tutorial](https://nvpro-samples.github.io/vk_raytracing_tutorial_KHR/vkrt_tutorial.md.html)
- [Ray Tracing In Vulkan](https://www.khronos.org/blog/ray-tracing-in-vulkan)