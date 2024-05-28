
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

To begin render pass, call `vkCmdBeginRenderPass`
For the last parameter of `vkCmdBeginRenderPass` we can set :
- `VK_SUBPASS_CONTENTS_INLINE`: The render pass commands will be embedded in the primary command buffer itself and no secondary command buffers will be executed.
- `VK_SUBPASS_CONTENTS_SECONDARY_COMMAND_BUFFERS`: The render pass commands will be executed from secondary command buffers.

## Basic Drawing commands

Finally, call `vkCmdBindPipeline` function to bind the graphics pipeline.
A second parameter of this function specifies if the pipeline object is a graphics or compute pipeline.

If we set states of viewport and scissors as dynamic states, we need to set them in the command buffer.

Then, issue draw command
`vkCmdDraw(commandBuffer, 3, 1, 0, 0);`
The parameters are :
- vertexCount 
- instanceCount - used for instanced rendering
- firstVertex - used as an offset into the vertex buffer(lowest value of `gl_VertexIndex`)
- firstInstance - used as an offset for instanced rendering




# Rendering and presentation

## outline of a frame

rendering a frame in Vulkan follows :
 - Wait for the previous frame to finish
 - acquire an image from swap chain
 - record a command buffer which draws the scene onto that image
 - submit the recorded command buffer
 - present the swap chain image

## Synchronization
The order of operations is up to us
-> using various sync primitives.(these tell the driver the order of operations)
Many Vulkan API are async -> the function will be returned before it finishes routine.

The things that we need to order explicitly are :
- acquire image from swap chain
- execute commands that draw onto the aqcuired one
- present that image to the screen for presentation

Two primitives we can use :
- semaphore 
- fences
Semaphore are used to specify the execution order of operations on GPU

semaphore for swap chain operation because they operates on GPU without making host wait.
fences for waiting previous frame because we need to make host wait in this case.

## Create Sync objects
`VkSemaphoreCreateInfo` , `VkFenceCreateInfo`
Should clean up these at the end of cleanup function.
## Waiting for the previous frame

we are going to specify another information on fenceInfo, which is `VK_FENCE_CREATE_SIGNALED_BIT` flag.
This flag makes a fence signaled state.
The reason that we need to set this flag on fence is because the very first frame doesn't have any previous frames and it must wait forever with unsignaled state of fence :(

## Acquiring an image from the swap chain

`vkAcquireNextImageKHR` : KHR postfix because swap chain is an extension feature.
params are :
- logical device
- swap chain
- timeout
- semaphore
- fence
- pointe of image index (used in frame buffer)

## Recording the command buffer

`vkResetCommandBuffer` to make sure it is able to be recorded
`recordCommandBuffer` to record the commands

## Submitting the command buffer

Queue submission and synchronization is configured through parameters
in the `VkSubmitInfo`
`VkSubmitInfo`
Parameters :
	- sType
	Below three parameters are for waiting on before execution begins and in which stage of the pipeline to wait.
	- waitSemaphoreCount
	- pWaitSemaphores : semaphores to be mapped with each stages that has same index in stage array.
	- pWaitDstStageMask : we need to define an stage array for this 
	Specify which command buffer to submit
	- commandBufferCount
	- pCommandBuffers

To submit command buffer to graphics queue, we use `vkQueueSubmit`
It takese parameters :
- VkQueue
- submitCount
- submitInfo
- optional fence that will be signaled when the command buffers finish execution


## Subpass dependencies
---
Subpass take care of image layout transitions.
These transitions are controlled by *subpass denpendencies*

subpass dendencies specify memory and execution dependencies between subpasses
-> note that the operations right before and after a single subpass also count as implicit "subpasses".

The right-before implicit subpass assumes that transition occurs at the start of the pipeline **although we haven't acquired the image yet** 
-> To handle this problem,
1. set a wait stage `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`
	Ensure that the render passes don't begin until the image is available.
2. set a wait stage `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT`
	Make render pass wait for the color attachment stage.

Subpass dendency are specified with `VkSubpassDependency`
Parameters are :
- srcSubpass
- dstSubpass
`dstSubpass` must always be higher than `srcSubpass` to avoid cycle in dependency graph
- srcStageMask : operations to wait on and the stages in which these operations occur.
	- To wait for the swap chain to finish reading from image before accessing it , we wait `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT` stage itself.
- srcAccessMask
- dstStageMask
- dstAccessMask

## Presentation
The last step is to submit the result back to the swap chain to show it up on the screen.
Presentation is configured through a `VkPresentInfoKHR`

## Frames in flight

How to stop waiting on the previous frame and make multiple frames to be in-flight?
Allow rendering of one frame to not interfere with the recording of the next.
-> need multiple command buffers, semaphores and fences.

With 2 frames in flight, CPU and GPU can be working on their own tasks at the same time.

Each frame should have its own command buffer.