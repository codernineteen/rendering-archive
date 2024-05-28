- Create a VkInstance
    
- Select a supported graphics card (VkPhysicalDevice)
    
- Create a VkDevice and VkQueue for drawing and presentation
    
- Create a window, window surface and swap chain
    
- Wrap the swap chain images into VkImageView
    
- Create a render pass that specifies the render targets and usage
    
- Create framebuffers for the render pass
    
- Set up the graphics pipeline
    Except for simple changes like viewport, any changes in graphics pipeline requires additional VKPipeline object.
    
- Allocate and record a command buffer with the draw commands for every possible swap chain image
    
- Draw frames by acquiring images, submitting the right draw command buffer and returning the images back to the swap chain (main loop)
	need for synchronization of execution order. (ex. semaphore, mutex.)


