
A new type of descriptor - combinded image sampler
- this enables us for shader to access an image through a sampler.

For this descriptor, we need to update how to create descriptors first.

---
# Update the descriptors

1. update layout binding
Originally we had only uniform buffer object binding in the descriptor set layout creation, but we need to add new set layout for sampler.

- create another descriptor sey layout binding for sampler
	- Note that stateFlags corresponded to fragment shader bit here.
- create an array containing the two layouts
- binding a pointer of the first element in the array to the layout creation info

2.  update descriptor pool
- expand a single pool size to an array  of pool sizes
- define another pool size for sampler (type is combined image sampler bit)
- set pointer of the array to `pPoolSizes` field of descriptor pool create info.
- 
Inadequate descriptor pools is a problem that validation layers cannot dectect. So we should handle descriptor pools carefully.


3. Finally, bind actual image and sampler resources to the descriptor.
- add `VkDescriptorImageInfo` into descriptor sets creation
- expand a single `VkWriteDescriptorSet` to an array of two descriptor infos
- specify a descriptor write struct about image info

---

# Texture coordinates

The key part of using texture is to map texture coordinates for each vertex.

- add new field 'texture coordinates' in Vertex struct
- add new element in an array of `VkVertexInputAttributeDescription`

---

# update shader code

- vertex shader
	- get a texture coordinate input using appropriate layout location
	- pass it to fragment shader
- fragment shader
	- get a texture coordinate from vertex shader
	- get texture sampler from layout binding
	- use built-in `texture` function to set output fragment color
