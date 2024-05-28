
In 3d application, meshes share vertices often.
![](../../../../../../images/Pasted%20image%2020240119093552.png)

To avoid this redundancy, we can use an index for a vertex so that we can reuse it between several triangles.

Just like the vertex data, we need to upload indices into a Vulkan buffer for GPU to be able to access them.

# Index Buffer creation
The creation of index buffer follows similar process to vertex buffer.
The only two differences are buffer size and `VkMemoryPropertyFlags` parameter which is set to `VK_BUFFER_USAGE_INDEX_BUFFER_BIT`

# Using an index buffer.
We need to add two more steps in recording command buffer.
- bind index buffer (we can have only a single index buffer)
	- `vkCmdBindIndexBuffer(commandBuffer, indexBuffer, bindingCount, indexType)`
- Change drawing command to tell Vulkan to use the index buffer.
	- `vkCmdDraw` -> `vkCmdDrawIndexed`