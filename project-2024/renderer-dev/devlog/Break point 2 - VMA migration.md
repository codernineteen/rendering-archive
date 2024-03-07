
In the previous implementation, i was allocating the memory of VkBuffer and VkImage by following vulkan tutorial, which separates the buffer creation into two different parts :
- buffer object creation
- memory allocation & binding

I thought this method would make it easy to make some mistakes if i don't keep track of all of the logics in the allocation always.
Also, There were some good usecase of VMA that improved frame rates of their project.

With these reason, i decided to delegate the hard part of memory allocation to VMA library.

While proceeding this tasks, i encountered numerous read access violation errors and it helped me a lot to understand Valid Vulkan API usage and memory allocation a lot deeper.

Then, let's step into this post from the break point

# Step 1 - include VMA

As vma documentation said, VMA is relatively easy-to-use library compared with other libraries that require building itself in Cmake compile chain.

I just need to download or copy the contents of 'vk_mem_alloc.h'.
I didn't need to set any compile definition to get vulkan functions in vk_mem_alloc implementation because i only needed the vulkan functions that are already linked statically in my project.

I just added this compile definiton.
```cpp
#define VMA_IMPLEMENTATION
```

# Step 2 - New type definition for VkBuffer
If we use `vmaCreateBuffer` for buffer creation and its memory allocation, the function fills out the given parameters `buffer`, `allocation` and `allocationInfo`.

Instead of managing this variables separately, i though it is good to handle them in a struct.
```cpp
#pragma once

#include <vulkan/vulkan.h>
#include <vma/vk_mem_alloc.h>

// a struct to wrap a Vkbuffer and its allocation from VMA
struct VkBufferAllocated 
{
	VkBuffer          buffer;
	VmaAllocation     allocation;
	VmaAllocationInfo allocationInfo;
};
```


# Step 3 - Modify buffer creation logic.

Before, i needed to check memory requirements, properties and other verbose stuff.
However, now i could leverage the power of VMA by removing all the verbose specifications
I just needed to specify a few more flags and 'memory usage' (not buffer usage).
Also if we add CREATE_MAPPED_BIT to allocation flags, we can directly access into mapped data at the struct member without need of calling 'vkMapMemory()'.

```cpp
VkBufferAllocated CreateBuffer(
	const VmaAllocator& allocator,
	VkDeviceSize size,
	VkBufferUsageFlags bufferUsage,
	VmaMemoryUsage memoryUsage,
	VmaAllocationCreateFlags memoryAllocationFlags
)
{
	// specify buffer creation info
	VkBufferCreateInfo bufferInfo = vkinfo::GetBufferCreateInfo(size, bufferUsage, VK_SHARING_MODE_EXCLUSIVE);

	VmaAllocationCreateInfo bufferAllocInfo{};
	bufferAllocInfo.usage = memoryUsage;
	/**
	* CREATE_MAPPED_BIT by default 
	*  - flag for automatic mapping pointer with 'pMappedData' member of VmaAllocationInfo struct (as long as buffer is accessible from CPU )
	*/
	bufferAllocInfo.flags = memoryAllocationFlags;

	// create buffer
	VkBufferAllocated newBuffer{};
	MK_CHECK(vmaCreateBuffer(allocator, &bufferInfo, &bufferAllocInfo, &newBuffer.buffer, &newBuffer.allocation, &newBuffer.allocationInfo));

	return newBuffer;
}
```


# Step 4 - Add Extensions

The most important thing that we should know when using Vulkan is the fact which Vulkan is platform-agnostic API.

Sometimes, I forgot this fact and use some functions that must be extended in advance without getting proxy address.
This behavior showed me a lot of linkage errors and i learned how to add appropriate extensions that i want to use.

Basically there are two extension types in Vulkan API.
1. instance extension
2. device extension

To check whether a extension is supported in relatively lower version of Vulkan API, we need to use different helper function based on the extension type.

## Instance extension
For instance one, we can call `vkEnumerateInstanceExtensionProperties`.
## Device extension
For device one, we can call `vkEnumerateDeviceExtensionProperties`.

To activate device extension, we need to set appropriate extension names in the vector and specify the information at the device creation time by link the vector with device creation info struct member (`ppEnabledExtensionNames`).

- I wanted to activate buffer device address extension to access buffer address of GPU device directly without passing through descriptor sets.
```cpp
const std::vector<const char*> deviceExtensions = {
	VK_KHR_SWAPCHAIN_EXTENSION_NAME,             // macro from VK_KHR_swapchain extension
	VK_KHR_BUFFER_DEVICE_ADDRESS_EXTENSION_NAME, // macro from VK_KHR_buffer_device_address extension
	VK_KHR_DEVICE_GROUP_EXTENSION_NAME, // macro from VK_KHR_device_group_creation extension
};
```


# Step 5 - Check an extension availability

To check whether 'buffer device address' feature is supported , i filled out `VkPhysicalDeviceFeatures2` with pNext pointer that point to `VkPhysicalDeviceBufferDeviceAddressFeatures`.

Here are some of things that i learned newly from this.
- Specify sType in advance without initialization.
	If we initialize a vulkan-specific struct, the members of it  are automatically filled with default value. This can lead to unexpected result.
	Also, if we don't fill sType field, Vulkan doens't know what the struct it should fill out now.
	To avoid above problems, we need to set sType by following Valid usage in Vulkan specification docs and don't initialize it.

- If there is no struct in pNext chaining anymore, we should set it as nullptr for the last struct in the chain.
	When we want to fill out some struct of properties and features using a function that fill out all of the fields based following pNext chain from the very first given struct, we should set pNext field as nullptr for the last struct in the chain.
	If we don't , we will encounter a lot of validation error messages.

```cpp
VkPhysicalDeviceProperties deviceProperties;
VkPhysicalDeviceFeatures2 deviceFeatures2;
VkPhysicalDeviceBufferDeviceAddressFeatures bufferDeviceAddressFeatures;

deviceFeatures2.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2;
bufferDeviceAddressFeatures.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_BUFFER_DEVICE_ADDRESS_FEATURES;

// pNext chain
deviceFeatures2.pNext = &bufferDeviceAddressFeatures; // attach buffer device address features to device features
bufferDeviceAddressFeatures.pNext = nullptr; // no more extension features

vkGetPhysicalDeviceProperties(device, &deviceProperties);
vkGetPhysicalDeviceFeatures2(device, &deviceFeatures2);
```

After querying the feaures information, we can check easily wheter buffer device address feature is supported with this statement.
```cpp
bufferDeviceAddressFeatures.bufferDeviceAddress != VK_TRUE
```


# Step 6 . Create a buffer with VMA

Now it's time to see an actual example of VMA usage for creating a buffer.

- Modified uniform buffer creation
	An uniform buffer will be updated in CPU side to update transformation matrices by user input.
	That's why i set a memory usage as `CPU_TO_GPU` explicitly.
	Then By `VMA_ALLOCATION_CREATE_MAPPED_BIT`, we can directly access in mapped data without calling `vkMapMemory` and memcpy to copy it to `void*` variable.
```cpp
void MKDescriptorManager::CreateUniformBuffer()
{
    // create uniform buffer object
    VkDeviceSize bufferSize = sizeof(UniformBufferObject);

    _vkUniformBuffers.resize(MAX_FRAMES_IN_FLIGHT);
    _vkUniformBuffersMapped.resize(MAX_FRAMES_IN_FLIGHT);

    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        _vkUniformBuffers[i] = util::CreateBuffer(
            _mkDevicePtr->GetVmaAllocator(),
            bufferSize,
            VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT,
            VMA_MEMORY_USAGE_CPU_TO_GPU,
            VMA_ALLOCATION_CREATE_MAPPED_BIT
        );
        _vkUniformBuffersMapped[i] = _vkUniformBuffers[i].allocationInfo.pMappedData;
    }
}
```


# Conclusion

This was a brief overview of Vulkan Memory Allocator usage for creating buffer !
I hope this documentation to help some people to struggle with specifying extensions and device features like me.