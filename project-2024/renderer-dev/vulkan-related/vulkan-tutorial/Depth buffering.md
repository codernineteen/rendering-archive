For complete 3d world, we need to add Z coordinate to the positions.

 We should change the format of attribute appropriately.
	 I added a slot for 'B' below
	`VK_FORMAT_R32G32_SFLOAT` -> `VK_FORMAT_R32G32B32_SFLOAT`.
But we can't see the right cube shape in the window.
![](../../../../images/Pasted%20image%2020240129152807.png)
This problem is caused by order of drawing.
To solve this problem, here are two solutions.
- sort all of draw calls by depth from back to front
- **use depth testing with a depth buffer.**

Depth testing enables us to solve order independency problem.
Every time the rasterizer produces a fragment, the depth test will check if the new fragment is closer than previous one.

Because glm library depth range is -1.0 ~ 1.0, we need to convert it into a range of 0.0 ~ 1.0 for Vulkan API.
```
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
```


# Depth image and view
---
A depth attachment is basically an image.(like color attachment).
-> we need image, memory and image view resources.

Creating a depth image requires :
Common parts are:
- same resolution as the color attachment defined by the swap chain extent
- image usage
- optimal tiling
- device local memory
and Unique part is
- depth specific format (indicated by `D??`)
	- `VK_FORMAT_D32_SFLOAT`: 32-bit float for depth
	- `VK_FORMAT_D32_SFLOAT_S8_UINT`: 32-bit signed float for depth and 8 bit stencil component
	- `VK_FORMAT_D24_UNORM_S8_UINT`: 24-bit float for depth and 8 bit stencil component

1. Query physical device **format properties** using `vkGetPhysicalDeviceFormatProperties`
	`VkFormatProperties` include :
	- `linearTilingFeatures`: Use cases that are supported with linear tiling
	- `optimalTilingFeatures`: Use cases that are supported with optimal tiling
	- `bufferFeatures`: Use cases that are supported for buffers
2. create an depth image with a function 'createImage' we wrote before.
	we need to add additional parameter on 'createImageView' about aspectFlags because depth image use `VK_IMAGE_ASPECT_DEPTH_BIT`
3. (optional) explicitly transition the depth image.


# Render pass
---
- Include depth attachment in render pass (need new `VkAttachmentDescription`).
- Fill out below struct
	- `VkattachmentDescription depthAttachment`
- Add a reference to the attachment for the first subpass
	- `VkAttachmentReference depthAttachmentRef`
	- `subpass.pDepthStencilAttachment = &depthAttachmentRef`
- Update subpass dependency (srcStageMask, srcAccessMask, dstStageMask, dstAccessMask)
- Update render pass info struct for `pAttachments` field to point to an array of two attachments


# Frame buffer
---
- Include depthImageView to attachments array to use depth testing every frame.
- modify `attachmentCount` and `pAttachments` fields properly

The color attachment differs for every swap chain image, but the same depth image can be used by all of them because only a single subpass is running at the same time due to our semaphores.


# Clear Values
---
Because we added another attachment in render pass and are using `VK_ATTACHMENT_LOAD_OP_CLEAR` bit in multiple attachments, we need to modify clear values in recording command buffer.

The range of depths in the depth buffer is 0.0(near view plane) to 1.0(far view place) in Vulkan.

For depth clearing,
- initial value is 1.0f , which is furthest possible depth
```cpp
std::array<VkClearValue, 2> clearValues{};
clearValues[0] = { {{0.0f, 0.0f, 0.0f, 1.0f}} }; // clear color
clearValues[1] = { 1.0f, 0 }; // clear depth
```


Note that order of clearValues should be identical order of our attachments which is set in render pass.


# Depth and stencil state
---
Depth testing needs to be enabled in the graphics pipeline.

Create `VkPipelineDepthStencilCreateInfo` struct and fill out fileds
```cpp
// depth and stencil testing
VkPipelineDepthStencilStateCreateInfo depthStencil{};
depthStencil.sType = VK_STRUCTURE_TYPE_PIPELINE_DEPTH_STENCIL_STATE_CREATE_INFO;
depthStencil.depthTestEnable = VK_TRUE;
depthStencil.depthWriteEnable = VK_TRUE;
depthStencil.depthCompareOp = VK_COMPARE_OP_LESS; // depth test passes if new depth is less than old depth
depthStencil.depthBoundsTestEnable = VK_FALSE; // optional
depthStencil.maxDepthBounds = 1.0f; // optional
depthStencil.minDepthBounds = 0.0f; // optional
// usage example : shadow volume
depthStencil.stencilTestEnable = VK_FALSE; // optional
depthStencil.front = {};
depthStencil.back = {};
```



- result
![](../../../../images/Pasted%20image%2020240129171909.png)


Remember destroying image and image view resource , then free device memory.