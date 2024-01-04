

# Window surface creation
Vulkan can't interface directly with the window system.
To make a connection between Vulkan and window system, we need to use WSI(Window System Integration) extensions.

There are some extensions for it, here we gonna use `VK_KHR_surface` (KHR suffix means that it is Khronos authored extension)

When we create an Vulkan Instance and related objects, we wrote this code
```cpp
std::vector<const char*> getRequiredExtensions() {
    uint32_t glfwExtensionCount = 0;
    const char** glfwExtensions;
    glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);

    std::vector<const char*> extensions(glfwExtensions, glfwExtensions + glfwExtensionCount);

    if (enableValidationLayers) {
        extensions.push_back(VK_EXT_DEBUG_UTILS_EXTENSION_NAME); // setup callback by using debug extension
    }

    return extensions;
}
```
Focusing on 'glfwGetRequiredInstanceExtensions', Although VkSurfaceKHR usage is platform agnostic, but its creation isn't because it depends on window-system details.
Regard to Windows OS, there is a platfom-specific addition to the extension 'VK_KHR_win32_surface'.
The extension is included in the list from 'glfwGetRequiredInstanceExtensions'.

- Create a member in class `VkSurfaceKHR surface;`
- Use glfwCreateWindowSurfcace function
```cpp
void createSurface() 
{ 
	if (glfwCreateWindowSurface(instance, window, nullptr, &surface) != VK_SUCCESS) { 
	throw std::runtime_error("failed to create window surface!"); 
	} 
}
```

# Querying for presentation support
Because we can't guarantee that every device in the system supports window system integration, we need to extend 'isDeviceSuitable' function to ensure that **a device can present images to the surface we created"

Presentation is a queue-specific feature.

- We need to add present queue family member in QueueFamilyIndices struct.
- test whehter a device support window system integration using `vkGetPhysicalDeviceSurfaceSupportKHR`

# Creating the Presentation Queue
