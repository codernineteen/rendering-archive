

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

## ref : platform specific extension usage
I will demonstrate how this platform specific extension can be used to create a surface on Windows, but we won’t actually use it in this tutorial. It doesn’t make any sense to use a library like GLFW and then proceed to use platform-specific code anyway. GLFW actually has `glfwCreateWindowSurface` that handles the platform differences for us. Still, it’s good to see what it does behind the scenes before we start relying on it.

To access native platform functions, you need to update the includes at the top:

```c++
#define VK_USE_PLATFORM_WIN32_KHR
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>
#define GLFW_EXPOSE_NATIVE_WIN32
#include <GLFW/glfw3native.h>
```

Because a window surface is a Vulkan object, it comes with a `VkWin32SurfaceCreateInfoKHR` struct that needs to be filled in. It has two important parameters: `hwnd` and `hinstance`. These are the handles to the window and the process.

```c++
VkWin32SurfaceCreateInfoKHR createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_WIN32_SURFACE_CREATE_INFO_KHR;
createInfo.hwnd = glfwGetWin32Window(window);
createInfo.hinstance = GetModuleHandle(nullptr);
```


The `glfwGetWin32Window` function is used to get the raw `HWND` from the GLFW window object. The `GetModuleHandle` call returns the `HINSTANCE` handle of the current process.

After that the surface can be created with `vkCreateWin32SurfaceKHR`, which includes a parameter for the instance, surface creation details, custom allocators and the variable for the surface handle to be stored in. Technically this is a WSI extension function, but it is so commonly used that the standard Vulkan loader includes it, so unlike other extensions you don’t need to explicitly load it.

```c++
if (vkCreateWin32SurfaceKHR(instance, &createInfo, nullptr, &surface) != VK_SUCCESS) {
    throw std::runtime_error("failed to create window surface!");
}
```


The process is similar for other platforms like Linux, where `vkCreateXcbSurfaceKHR` takes an XCB connection and window as creation details with X11.

# Querying for presentation support
Because we can't guarantee that every device in the system supports window system integration, we need to extend 'isDeviceSuitable' function to ensure that **a device can present images to the surface we created"

Presentation is a queue-specific feature.

- We need to add present queue family member in QueueFamilyIndices struct.
- test whehter a device support window system integration using `vkGetPhysicalDeviceSurfaceSupportKHR`

# Creating the Presentation Queue
