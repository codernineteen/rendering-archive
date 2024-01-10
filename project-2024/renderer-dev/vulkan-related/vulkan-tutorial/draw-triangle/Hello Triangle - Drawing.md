
# Framebuffers
---
Things to know :
- an image acquired from swap chain
	- swap chain is a colleciton of render targets.
- wrapping attachments into `VkFramebuffer` object.
- a framebuffer references all of the `VkImageView` objects
- `VkImageView` references part of an image. ( image <- vk image view <- vk frame buffer)
- the image that we have to use for attachement depends on 'which image the swap chain returns when we retrieve one'
	-> eventually, we have to create a framebuffer for all of images in the swap chain.

How to create :
- resize `std::vector<VkFramebuffer>` to fit into size of swap chain image views.
- iterate image view and create each frame buffer from them.
	- specify (render pass, attachment information, width and height, layers)

# Command buffers
Actual drawing commands starts!!

Type of commands :
- drawing operations
- memory transfers
-> these are not executed by function calls.

Instead, we records it in command buffer objects to operate them.

Advantage of command buffers :
- all of commands submitted together -> Vulkan process the commands more efficiently.
- allows command recording to happen in multiple threads.

## Command pool
---
Create command pool first before creating buffers.
This pool manage the memory used to store the buffers and command buffers is allocated to the space.

`VkCommandPoolCreateInfo`
- sType
- flags :
	-  `VK_COMMAND_POOL_CREATE_TRANSIENT_BIT`: Hint that command buffers are rerecorded with new commands very often (may change memory allocation behavior)
	- `VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT`: Allow command buffers to be rerecorded individually, without this flag they all have to be reset together
- queueFamilyIndex :
	- **Each command pool can only allocate command buffers that are submitted on asingle type of queue.**


## Command Buffer allocation
---
Command buffer is automatically destroyed.
Use `vkAllocateCommandBuffers` to allocate memory for command buffers.
this funciton takes `VkCommandBufferAllocateInfo` sturct as parameter.

`VkCommandBufferAllocateInfo`
- sType
- commandPool
- level : specify whether the buffers are primary or secondary command buffers
	- `VK_COMMAND_BUFFER_LEVEL_PRIMARY`: Can be submitted to a queue for execution, but cannot be called from other command buffers.
	- `VK_COMMAND_BUFFER_LEVEL_SECONDARY`: Cannot be submitted directly, but can be called from primary command buffers. -> **useful for reusing common operations in primary command buffer**

## Command Buffer recording
---
Writes commands we want to exectue into a command buffer.

Use Two parameters
- command buffer
- index of the current swapchain image.

Recording always starts with `vkBeginCommandBuffer` with `VkCommandBufferBeginInfo` struct.

`VkCommandBufferBeginInfo`
- sType
- flags :
	- `VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT`: The command buffer will be re-recorded right after executing it once.
	- `VK_COMMAND_BUFFER_USAGE_RENDER_PASS_CONTINUE_BIT`: This is a secondary command buffer that will be entirely within a single render pass.
	- `VK_COMMAND_BUFFER_USAGE_SIMULTANEOUS_USE_BIT`: The command buffer can be resubmitted while it is also already pending execution.
- pInheritanceInfo :
	This field only releted to secondary command buffers.
	Specify which state to inherit from the calling primary command buffers.

Note : If the command buffer was already recorded once, then a call to `vkBeginCommandBuffer` will implicitly reset it. It’s not possible to append commands to a buffer at a later time.
## Starting a render pass

Drawing starts with `vkCmdBeginRenderPass` with `VkRenderPassBeginInfo` struct.

`VkRenderPassBeginInfo`
Below three parameters represent render pass itself.
- sType
- renderpass
- framebuffer :
	- a frame buffer create for a swap chain image specified as a color attachment.
These define the size of render area. We should match the size of attachments for peformance.
	The render area defines where shader loads and stores will take place.
- renderArea.offset
- renderArea.extent 
Lastly, define clear values relate to `VK_ATTACHMENT_LOAD_OP_CLEAR`
- clearValueCount
- pClearValues : set pointer of `VkClearValue` here.