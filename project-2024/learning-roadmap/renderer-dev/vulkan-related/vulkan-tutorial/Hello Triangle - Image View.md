
To use `VkImage` included in the swap chain, we have to create `VkImageView` in render pipeline.
image view describes :
- how to access the image and which part to access


# VkImageViewCreateInfo
fields are : 
- sType - structure type
- image - an image in the swap chain
- viewType - how to treat images(1d, 2d, 3d textures or cube maps)
- format - swap chain image format
- components - swizzle color channels
- subresourceRange - image's purpose, which part should be accessed
	  -> we can set multiple layers here when we develop stereographic 3D app
	