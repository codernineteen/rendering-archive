
## Resource management

 In cpp, we can mangage memory that is dynamically allocated with [RAII](https://en.cppreference.com/w/cpp/language/raii) pattern.
 when we create Vulkan objects, we use 'vkCreateXXX' or 'vkAllocateXXX' provided by the api. It is trivial that we should destroy the resources allocated by the functions using counter-part destroyer like 'vkDestroyXXX'.
 To achieve this, we can expand standard smart pointers or build our new custom allocators.


## Vk instance
This instance makes a connection between our application and vulkan library
To give useful information to driver, we can fill out application information properties in 'VKApplicationInfo' struct.
There is a 'pNext' field in VKxxx struct which can point to extension information.

A lot of information in Vulkan is passed through another structs.
For example,
```cpp
void createInstance() {
	VkApplicationInfo appInfo{};
	appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO; // set app info enum variant to struct Type field
	appInfo.pApplicationName = "VKRenderer"; // application name
	appInfo.applicationVersion = VK_MAKE_VERSION(1,0,0); // version specifier
	appInfo.pEngineName = "NO ENGINE";
	appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
	appInfo.apiVersion = VK_API_VERSION_1_3; // VKRenderer using sdk version 1.3.268

	
	VkInstanceCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
	createInfo.pApplicationInfo = &appInfo;
}

```


Because Vulkan is a platoform-agnostic(independent) API, we need to give additional extension about window system.

## VKResult
Without a need to store results of processing vulkan routine, we can just validate the result of routine with a type 'VKResult'. (quite similar to rust programming language's Result haha)
When it was successful, the type will be marked as VK_SUCCESS.
```cpp
if (vkCreateInstance(&createInfo, /*custom allocator*/ nullptr, &instance) != VK_SUCCESS) {
    throw std::runtime_error("failed to create an instance");
}
```


## Validation layer

Using vulkan means that there are lots of things that developer should handle.
For example, using a new GPU feature and forgetting to request it at **logical device** creation time.
To avoid errors hard to debug from mistakes, we can use elegant system called as 'validation layers'

Key features in validation layer
-  Checking the values of parameters against the specification to detect misuse
    
- Tracking creation and destruction of objects to find resource leaks
    
- Checking thread safety by tracking the threads that calls originate from
    
- Logging every call and its parameters to the standard output
    
- Tracing Vulkan calls for profiling and replaying

Validation layers can only be used when it is installed in host system. so we need to add a feature to turn off validation layer selectively when we release it.


[Two types of validation layers]
1. instance specific : check calls related to global vulkan objects
2. device specific : the layer is deprecated now, but it is recommended to enable it for compatibility.

# Physical device

After initializing Vulkan library(instance), we need to look for and select a graphics card in the system. (Note that we can select graphic cards as many as we want.)

1. Enumerate physical devices in system
2. Check suitability of each devices using vkGetXXX API
	- vkGetPhysicalDeviceProperties
	- vkGetPhysicalDeviceFeatures
- `VK_PHYSICAL_DEVICE_TYPE_OTHER` - the device does not match any other available types.
    
- `VK_PHYSICAL_DEVICE_TYPE_INTEGRATED_GPU` - the device is typically one embedded in or tightly coupled with the host.
    
- `VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU` - the device is typically a separate processor connected to the host via an interlink.
    
- `VK_PHYSICAL_DEVICE_TYPE_VIRTUAL_GPU` - the device is typically a virtual node in a virtualization environment.
    
- `VK_PHYSICAL_DEVICE_TYPE_CPU` - the device is typically running on the same processors as the host.

We can also use score policy to pick a physical device.


# Queue Families
There are many queue families in Vulkan and each family of queue allows only  a subset of commands.
ex. a family that allows processing of compute commands, a family that allows memory transfer.

Before looking for another queue family, it is good choice to define indices struct in advance.
Notice that i wrapped uint32_t with '[std::optional](https://en.cppreference.com/w/cpp/utility/optional)'. `std::optional` is a wrapper that contains no value until we assign something to it.
```cpp
struct QueueFamilyIndices {
	std::optional<uint32_t> graphicsFamily;
};

QueueFamilyIndices findQueueFamilies(VkPhysicalDevice device) {
    QueueFamilyIndices indices;
    // can detect assignment from now on.

    return indices;
}
```


we need to find at least one queue family that supports `VK_QUEUE_GRAPHICS_BIT`
```cpp
QueueFamilyIndices findQueueFamilies(VkPhysicalDevice device) {
    QueueFamilyIndices indices;

    uint32_t queueFamilyCount = 0; // query number of queue families
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, nullptr);

    std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data()); // get all queue families

    int i = 0;
    for (const auto& queueFamily : queueFamilies) {
        if(queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT) { 
   			indices.graphicsFamily = i; // assign an index to graphicsFamily queue
       	}

        i++;
    }

    return indices;
}
```


# Logical device
Now it's time to make an interface of a physical device , which is called as 'logical device'.

- Initialize VkDeviceQueueCreateInfo
	 This structure describes the number of queues we want for a single queue family
 - Deternine queue priority(ranged between 0.0 ~ 1.0)
	   This influence the scheduling of command buffer execution 
- Initialize main `VkDeviceCreateInfo` structure
	- link queue info struct to 'pQueueCreateInfos' member

Logical devices don’t interact directly with instances, so that's why it’s not included as a parameter in logical device creation function.


# Retrieving queue handle

The queues are automatically created with logical device. so we just need to use API to get them.
1. define a variable to store queue handle . ex) `VkQueue graphicsQueue`.
2. use get api to retrieve the queue. ex) `vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);`

With the logical device and queue handles we can now actually start using the graphics card to do things!