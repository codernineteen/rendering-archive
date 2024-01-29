
Adding a texture follow the steps :
- create an image object backed by device memory (query memory requirement and allocate device memory)
- fill it with pixels from an image file **explicitly**
- create an image sampler
- add a combined image sampler descriptor to sample colors from the texture.

Creating images is similar to create buffers.
However, a key difference here :

- images can have different layouts
	- `VK_IMAGE_LAYOUT_PRESENT_SRC_KHR`: Optimal for presentation
	- `VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL`: Optimal as attachment for writing colors from the fragment shader
	- `VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL`: Optimal as source in a transfer operation, like `vkCmdCopyImageToBuffer`
	- `VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL`: Optimal as destination in a transfer operation, like `vkCmdCopyBufferToImage`
	- `VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL`: Optimal for sampling from a shader

One of the most common ways to transition the layout of an image is *pipeline barrier* (sycnrhonization on image write, transition layouts and  so on)

# Staging buffer
Like we've done for vertex buffer before, we create host-visible memory first and copy the data from pixels we got from image (To load an image, we can use 'stb_image.h' library - https://github.com/nothings/stb/blob/master/stb_image.h)


# Texture image
It is better to use image objects to access the pixel values. (faster retrieve and easier than setting up shader)

We need these two variables
```cpp
VkImage textureImage; 
VkDeviceMemory textureImageMemory;
```

To create an image object , use `VkImageCreateInfo`
Parameters :
- sType
- imageType
- extent - specifies the dimensions of the image (how many texels are there on each axis)
	- width
	- height
	- depth
- mipLevels
- arrayLayers
- format - we should use same format for the texels as the pixels in the buffer. (For the case when graphics hardware doesn't support the format, we need to prepare for alternative formats)
- tiling - specify texel layout
	- `VK_IMAGE_TILING_LINEAR`: Texels are laid out in row-major order like our `pixels` array
	- `VK_IMAGE_TILING_OPTIMAL`: Texels are laid out in an implementation defined order for optimal access
- initialLayout 
	- `VK_IMAGE_LAYOUT_UNDEFINED`: Not usable by the GPU and the very first transition will discard the texels.
	- `VK_IMAGE_LAYOUT_PREINITIALIZED`: Not usable by the GPU, but the first transition will preserve the texels. -> this may be required for transition the image to be a transfer source without loss of data.
- usage
- samples : related to multisampling
- flags : optional flags (such as sparse image in voxel volume)

# Layout transitions

1. create image memory barrier
`VkImageMemoryBarrier`
- sType
- oldLayout
- newLayout
- srcQueueFamilyIndex - for transferring queue ownership 
- dstQueueFamilyIndex - same reason
	- VK_QUEUE_FAMILIY_IGNORED not to transfer ownership
- subresourceRange
- srcAccessMask
- dstAccessMask

Then, call `vkCmdPipelineBarrier`.
We can specify a stage before barrier and another stage that will wait the stage in the parameter set. (refer to [this table](https://www.khronos.org/registry/vulkan/specs/1.3-extensions/html/chap7.html#synchronization-access-types-supported))
ex)
If we want to read from uniform after the barrier,
`VK_ACCESS_UNIFORM_READ_BIT` -> `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT`(wait read bit)