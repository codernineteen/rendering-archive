- Example of mipmaps
![mipmap example](https://docs.vulkan.org/tutorial/latest/_images/images/mipmaps_example.jpg)

Mipmaps are pre-computed downscaled versions of original image.
The each of downscaled version has half of width and height of previous image.

When an object is far away from away, it don't need to render the object with original image. Instead of doing so, we can use downscaled-image version related to a distance between the object and eye.

1. we need to fill out member variable related to mipmaps properly
	- check out `VkImage`, `VkImageView` and `VkImageMemoryBarrier`

2. Although texture image has multiple mip levels, staging buffer only can be used to fill mip level 0.
	To fill other levels, we need to generate the data from the single level.
	-> use `vkCmdBlitImage`
		this is a transfer operation , so we must inform Vulkan that we are going to use the texture image as **both src and dst of a transfer.**
		Also, blit operation depends on the layout of the image. For better performance, use `VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL, DST_OPTIMAL` bit for layouts.
3. modify transition image layout
	- originally we peforms layout transitions on the entire image
	- Now we need to leave each level of texture image in `VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL` -> blit command reading from dst -> `VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL`

A generate mipmaps function does :
- Basically reuse `VkImageMemoryBarrier` and change `subresourceRange.miplevel`, `oldLayout`, `newLayout`, `srcAccessMask` and `dstAccessMaks` in each transition.

- Start a loop to record each of `vkCmdBlitImage` commands (iterator starts at 1, not 0)
	1.  baseMipLevel `i-1` -> `VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL`
	  This transition will wait for level `i-1` to be filled(from previous blit command or `vkCmdCopyBufferToImage`)
	2. specify blit comand
		- specify a region used in the blit operation
		- specify source mip level and destination mip level
		- the data will be blitted from `srcOffsets` to `dstOffsets`
		- `dstOffsets[1]` specify X and Y dimenstion depending on previous mip level (if mipWidth is smaller than 1, just set 1)
			Also, Z dimension is set to 1 because 2d image has a depth of 1.
	3. record blit command(`vkCmdBlitImage`)
		- same texture image is used for both of `srcImage` and `dstImage` parameter.
		- source mip level -> `VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL` and destination level is still in `VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL`
	4. After a blit command(SRC to DST), we need to transition the SRC to SHADER_READ_ONLY so that fragment shader can read it.
		Set proper memory barrier for this and it will wait on the previous blit transition to finish.
	5. For last iteration in loop, we need to transition DST_OPTIMAL to SHADER_READ_ONLY layout additionally because this won't be handled by the loop


# Linear filtering support
---

`vkCmdBlitImage` is not supported on all platforms. (this command requires linear filtering support.)

To check if a platform support linear filtering, we can call `vkGetPhysicalDeviceFormatProperties` function and the function will fill out `VkFormatProperties` struct

`VkFormatProperties`
Each field describes how the format can be used depending on the way it is uesd.
- `linearTilingFeatures`
- `optimalTilingFeatures`
- `bufferFeatures`

If a platform doesn't support linear filtering, there are two alternatives.
1. implement a function that searches common texture image formats for the platform which doesn't support linear blitting
2. implement mipmap generation

# Sampler
---
- `VkImage` : holds mipmap data
- `VkSampler` : controls how the data is read while rendering.
	  When a texture is sampled, the sampler selects a mip level.
	- If `VK_SAMPLER_MIPMAP_MODE_NEAREST` is set to mipmapMode, level of details select the mip level to sample from
	- If `~_LINEAR` , level of details select two mip levels to be sampled. (for blending)
	Then, when lod(level of detail) is lower or equal to zero(the minimum value of lod is zero in general), It means that object is close to camera and we use magFilter.
	On the other hand, we use minFilter for objects far away from camera.