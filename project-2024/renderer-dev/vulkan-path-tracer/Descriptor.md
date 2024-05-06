
- Descirptor is the only way to read and write referencing data in shader. (similar to pointer)

- it can reference subranges of resources (ex. range of data in a buffer)
- application can bind descriptor sets all at once through a set of structs `VkDescriptorSet`
- Each set can be allocated from `VkDescriptorPool`


- Being similar to a relationship between c++ function and its function signature,
  a Vulkan pipeline specify the types of descriptor that it use with `VkDescriptorSetLayout` 
	  A descriptor set layout has several bindings to itself.

The below image is an overview of how descriptor works.
![](../../../images/Pasted%20image%2020240505205651.png)


with nvvk, there is a `nvvk::DescriptorSetContainer` that makes the process of creating descriptor shorter.
- dependency graph
  ![](../../../images/Pasted%20image%2020240505205952.png)


A slight changes are required in the order of command execution :
- **begin command buffer**
- bind compute pipeline point
- **bind descriptor set to the pipeline**
- dispatch the pipeline with workgroups for the entire image.
- **end command buffer**

