It is possible for the window surface to change such that the swap chain is no longer compatible with it.
 -> one of reason that causes this  is 'resizing window'
 -> to prevent this, we need to detect these events and recreate the swap chain.


1. Wait logical device until it becomes idle state
2. destroy old swap chain resource
3. recreate swap chain, image views and frame buffers (also, render pass if it is required.)
	**No need to modify choosing extent** because we query the new window resolution to make sure that the **swap chain images** have the right size

We have still a problem that we need to stop all rendering before creating the new swap chain.
we can create a new swap chain while drawing commands on an image from old-swap chain is still in-flight.
If we want to avoid this problem, we need to set oldSwapchain field to `VkSwapchinCreateInfoKHR` struct.


# Suboptimal or out-of-date swap chain

`vkAcquireNextImageKHR` and `vkQueuePresentKHR` functions return special valuse to detect events that causes swapchain recreation.

return values are :
- `VK_ERROR_OUT_OF_DATE_KHR`: The swap chain has become incompatible with the surface and can no longer be used for rendering. Usually happens after a window resize.
- `VK_SUBOPTIMAL_KHR`: The swap chain can still be used to successfully present to the surface, but the surface properties are no longer matched exactly.

# Avoid deadlock
Fences can be unsignaled on the host with `vkResetFences`.
In previous drawFrame function, we reset fences right away after finished waiting.
But from OUT_OF_DATA_KHR error, if we return earlier without submitting anything on queue, the fence will never be signaled. Therefore, Deadlock!!

we can modify the code like this.
```cpp
vkWaitForFences(device, 1, &inFlightFences[currentFrame], VK_TRUE, /*timeout*/UINT64_MAX); // wait previous frame to finish

uint32_t imageIndex;
// used semaphore to wait presentation engine to finish presentation here.
VkResult result = vkAcquireNextImageKHR(device, swapChain, /*timeout*/UINT64_MAX, imageAvailableSemaphores[currentFrame], VK_NULL_HANDLE, &imageIndex);

if (result == VK_ERROR_OUT_OF_DATE_KHR) {
    recreateSwapChain();
    return;
}
else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR) {
    throw std::runtime_error("failed to acquire swap chain image!");
}

// only reset the fence if we are submmitting work to avoid deadlock
vkResetFences(device, 1, &inFlightFences[currentFrame]); // reset fence to unsignaled state
```


# Handling resizes explicitly
`VK_ERROR_OUT_OF_DATE_KHR` is not always guaranteed to be triggered over resizing window.
-> we need to handle this explicitly either.


- set a member variable to determine resize event
- set an arbitrary pointer in the resize callback using `glfwSetWindowUserPointer`