swap chain - an infrastructure that will own the buffers that we render to
It is a queue of images that are waiting to be presented to the screen.

# Check support first
- Not all graphics card support direct presentation to a screen
- image presentation heavily tied into the window system
	-> have to enable `VK_KHR_swapchain`

1. Create device extension name vector first(can use macro)
2. check device extension support
	- enumerate device extension properties and get a count
	- fill available extensions vector
	- compare available list with device extension name that we want to check

# Enable device extensions
To use swap chain, we need to enable the VK_KHR_swapchain extension first.
we just need to add a few of informations in logical device createInfo structure.


# Querying details of swap chain support

After check whether a swap chain extension is available, we need to check if it is compatible with window surface.

**Check list**
- Basic surface capabilities - max/min width and height, # of images
- Surface formats - pixel, color space
- Available presentation modes


When we query each of details using API, we need to pass 'physical device' and 'surface' struct because they are the core components of swap chain.


# Choosing the right settings for the swap chain
Three types of settings to determine :
 - Surface format (color depth)
	 We need to pick each of 
	 - color channels
	 - color space
 - **Presentation mode (conditions for "swapping" images to screen)** <- most important
	 - VK_PRESENT_MODE_IMMEDIATE_KHR 
		   an image transferred to the screen right away.
	 - VK_PRESENT_MODE_FIFO_KHR
		 the queue works as fifo manner. if the queue is full, need to wait.
	 - VK_PRESENT_MODE_FIFO_RELAXED_KHR
		 Instead of waiting, the image is transferred right wasy when it finally arrives
	 - VK_PRESENT_MODE_MAILBOX_KHR
		 the images that are already queued are simply replaced with the newer ones.
 - Swap extent (resolution of images in swap chain)
	 - possible resolution is defined in `VkSufaceCapabilitiesKHR`
	 - match resolution of window by setting the width and height in the `currentExtent`
	 - some window managers allow us to set special value
		 -> maximum value of uint32_t.
		 In this case, find best matches within `minImageExtent` and `maxImageExtent`

**spec reference**
Note
For reference, the mode indicated by `VK_PRESENT_MODE_FIFO_KHR` is equivalent to the behavior of {wgl|glX|egl}SwapBuffers with a swap interval of 1, while the mode indicated by `VK_PRESENT_MODE_FIFO_RELAXED_KHR` is equivalent to the behavior of {wgl|glX}SwapBuffers with a swap interval of -1 (from the {WGL|GLX}_EXT_swap_control_tear extensions).

Find the best one first, and if not, find next one 

# Create swap chain
