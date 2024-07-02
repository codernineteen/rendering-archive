
Memory access from CPU may not be the most optimal choice.
Two options :
- staging buffer : CPU accessible memory to upload data from the vertex array
- device local memory : final vertex buffer.
Then use a buffer copy command to move the data.(staging buffer -> actual vertex buffer.)

# Explicit transfer queue
Buffer copy command requires a queue family that supports transfer operations.
Fortunatyl, any queue with  a flag `VK_QUEUE_GRAPHICS_BIT` or `VK_QUEUE_TRANSFER_BIT` support the transfer operations implicitly.

we can use a different queue family for transfer operations and turn on 'sharing mode' against resources. -> requires another command pool because we need to submit any transfer commands to there 

The steps mentioned in tutorial
- Modify `QueueFamilyIndices` and `findQueueFamilies` to explicitly look for a queue family with the `VK_QUEUE_TRANSFER_BIT` bit, but not the `VK_QUEUE_GRAPHICS_BIT`.
    
- Modify `createLogicalDevice` to request a handle to the transfer queue
    
- Create a second command pool for command buffers that are submitted on the transfer queue family
    
- Change the `sharingMode` of resources to be `VK_SHARING_MODE_CONCURRENT` and specify both the graphics and transfer queue families
    
- Submit any transfer commands like `vkCmdCopyBuffer` (which we’ll be using in this chapter) to the transfer queue instead of the graphics queue


# Abstracting buffer creation


Before creating multiple buffer, we can abstract this process into a function because creating multiple buffers share some common parts.

Add **buffer size**, **memory properties** and **usage** so that we can make this function flexible for any type of buffer creation.

# Staging buffer
Now we're going to use host-visible buffer as temporary buffer(which is the same buffer type we've used so far) and use a device local one as actual vertex buffer.

For device local, we need bit masks
- Destination bit
- vertex vuffer bit
- device local bit
```cpp
createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, vertexBuffer, vertexBufferMemory);
```

In quick summary,
stage buffer -> for mapping and copying the vertex data
device local buffer -> copy from stage buffer(using SRC/DST flag) and works as actual vertex buffer to be used in graphics card.

**Memory transfer operations** are executed using **command buffers** like drawing.
-> need to allocate temporary command buffer first 
-> may be better to use another command pool for this short-lived command buffers.

1. allocate Command buffer with creation info into a certain command pool
2. begin command buffer to record commands
3. `vkCmdCopyBuffer` to tranfer contents of buffers. (need offsets of src, dst buffer each)

To wait transfer to complete, we can use fence primitive with `vkWaitForFences` or `vkQueueWaitIdle`.
Fences give us more opportunities to optimize when multiple tranfers occur.

We should not forget to destroy buffer and memory aafter copying the data from staging buffer.

# Further discussion
The right way to allocate memory for a large number of objects at the same time is to create a custom allocator that splits up a single allocation among many different objects by using the `offset` parameters that we’ve seen in many functions.
https://developer.nvidia.com/vulkan-memory-management