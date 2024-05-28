Create a actual descriptor set for each `VkBuffer` resource to bind it to the uniform buffer descriptor

# Descriptor pool

Descriptor sets must be allocated from a pool like command buffers.

`VkDescriptorPoolSize`
- type : descriptor type
- descriptorCount  

`VkDescriptorPoolCreateInfo`
- sType
- poolSizeCount 
- pPoolSizes
- maxSets


# Descriptor set
`VkDescriptorSetAllocateInfo`
- sType
- descriptorPool
- descriptorSetCount
- pSetLayouts

We don't need to cleanup the sets explicitly, but need to do for layouts and pool.

After allocating descriptor sets, we need to populate every descriptor in a loop.
Descriptors that refer to buffers are configured with a `VkDescriptorBufferInfo`

`VkDescriptorBufferInfo`
- buffer : a VkBuffer mapped to this descriptor
- offset 
- range : size of buffer object

Then, we need `VkWriteDescriptorSet` struct to update descriptor sets.
`VkWriteDescriptorSet`
 - sType
 - dstSet :  a descriptor set as destination
 - dstBinding
 - dstArrayElement : descriptors can be arrays -> need to specify the first index in the array that we want to update. If not array, simple set '0'
 - descriptorType : we need to specify the type again although we set it in layout because it is possible to update multipe descriptors in an array.
 - descriptorCount : how many elements in array we want to update.
 - pBufferInfo
 - pImageInfo
 - pTexelBufferView

After specifying all the informations, call `vkUpdateDescriptorSets` to update descriptors sets configuration.

# Using descriptor sets
update `recordCommandBuffer` to bind the right descriptor set for each frame to the descriptors in the shader.
call `vkCmdBindDescriptorSets` before `vkCmdDrawXXX`
Descriptor sets is not unique, so we need to determine if we want to bind graphics pipeline or compute pipeline.

# Alignment requirement
By using the same struct types, we've matched c++ struct with uniform definition in shader.
However,  If we don't care about the memory alignment in the struct, we may get unexpected result.

For example, adding a new field 'vec2' will cause a problem. 
```cpp
// From
struct UniformBufferObject 
{ 
 glm::mat4 model; // offset : 0
 glm::mat4 view;  // offset : 64bytes
 glm::mat4 proj;  // offset : 128bytes
};
// To
struct UniformBufferObject 
{  
 glm::vec2 foo; // offset : 0
 glm::mat4 model; // offset : 8bytes 
 glm::mat4 view;  // offset : 72bytes;
 glm::mat4 proj;  // offset : 136 bytes 
 // none of model,view and proj are multiplies of 16
};
```

Vulkan expects the data in our struct to be aligned in memory with this way example
- Scalars have to be aligned by N (= 4 bytes given 32 bit floats).
- A `vec2` must be aligned by 2N (= 8 bytes)
- A `vec3` or `vec4` must be aligned by 4N (= 16 bytes)
- A nested structure must be aligned by the base alignment of its members rounded up to a multiple of 16.
- A `mat4` matrix must have the same alignment as a `vec4`.

To fit this alignment requirement, we can take an advantage of `alignas` introduced in C++11
ex)
```cpp
struct UniformBufferObject {
    glm::vec2 foo;
	alignas(16)glm::mat4 model;
	glm::mat4 view;
	glm::mat4 proj;
};
```
Or we can define `GLM_FORCE_DEFAULT_ALIGNED_GENTYPES` to force a version of vec2 and mat4 that has the alignment requirements already specified for us.

But it doesn't prevent the case when a nested struct messep up alignment in outer struct.
In this case, we must specify the alignement using `alignas` ourselves.

In conclusion, it is always safe to be explicit about alignment.
```cpp
struct UniformBufferObject {
	alignas(16)glm::mat4 model;
	alignas(16)glm::mat4 view;
	alignas(16)glm::mat4 proj;
};
```