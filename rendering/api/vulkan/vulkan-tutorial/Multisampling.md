
When we rendered a quad in the middle of tutorial, we saw there were aliasing at the boundaries of the quad.
![aliasing](https://docs.vulkan.org/tutorial/latest/_images/images/texcoord_visualization.png)
This is because we have limited number of pixels to represent a perfect line.

This is how lines determined by vertices are converted into pixels.
A pixel color is determined based on a single sample point.
![](https://docs.vulkan.org/tutorial/latest/_images/images/aliasing.png)

But If we use multi-sampling anti-aliasing, shortly MSAA, it uses multiple sample points per pixel to deteminte final color of the pixel.
![MSAA](https://docs.vulkan.org/tutorial/latest/_images/images/antialiasing.png)


# Get available sample counts
To get maximum number of samples , call `vkGetPhysicalDeviceProperties` and fill out `VkPhysicalDeviceProperties` by passing its reference to the function call.

The informations is included in `limits` field. 
we need to find the highest common count supported by color and depth buffer.
`physicalDeviceProperties.limits.framebufferColorSampleCounts & physicalDeviceProperties.limits.framebufferDepthSampleCounts;`

Then, update MSAA sample counts **at the physical device picking time.**

# Setting up render target
In MSAA, 
each pixel sampled in offscreen buffer(store more than one sample per pixel) 
-> rendered to screen(default frame buffer - only a single sample per pixel).

we need an additional render target for above process.
This image will have to store the desired number of samples per pixel and the information should be specified at the creation time(`VkImageCreateInfo`).
```cpp
VkImage colorImage;
VkDeviceMemory colorImageMemory;
VkImageView colorImageView;
```

```cpp
imageInfo.samples = numSamples;
```

we abstracted a creation of `VkImage` into a function.
Now we need to modify `samples` field setting in the function by getting a related parameter.
Also, we should destroy the color buffer resource when we clean up swap chain and need to recreate whenever swapchain recreation is required.

# New attachments
First, Our original color attachment was using `VK_IMAGE_LAYOUT_PRESENT_SRC_KHR`, but now we can present images directly because of MSAA. 
-> change it into `VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL`

Then we need to add only one attachemnt more because we don't need one for depth buffer because it is not presented anywhere.

We will use the original color attachment for sampling purpose and the new attachment will take a role to present final image, whose layout is `VK_IMAGE_LAYOUT_PRESENT_SRC_KHR`.

After then,
- create a attachment ref and link it to resolve attachment field in subpass struct
- change `srcAccessMask` field in subpass dependency.
 
# Reference
https://docs.vulkan.org/tutorial/latest/10_Multisampling.html