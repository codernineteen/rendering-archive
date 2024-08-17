
# 1. Introduction
---

While i was developing a feature to support GLTF file format for physically based shading later, I needed to write too many verbose specification for render passes of each `VkPipeline`.

Meanwhile, i found an extension whose name is `VK_KHR_dynamcic_rendering` from vk-guide tutorial.
The extension discards every unintuitive specifications for render pass and target frame buffers. At the same time, it requires explicit memory pipeline barrier command call that a render pass implicitly perform.
I thought the dynamic rendering feature is more close to Vulkan design principle because it need to transition image layout appropriately to see correct result on screen.
Also, The render pass concept was one of the most complex things to me and i couldn't understand which relationship exists between render pass and framebuffer.
With these reasons, This was a great chance to migrate rendering logic from original render pass to this extension before i start to write additional pipelines in the near future.

Let's see how to enable this extension in Vulkan application and its actual usages!

# 2. Enable device extension

The extension is a type of device extension.
To enable this extension, we need to write some codes at device creation time.

From now on, all code snippets is specific to my application design. So If you have your own application, it may helps you to focus only the order of implementation steps.

I have device extension name list and specified the extension name there.
```cpp
public:
	const std::vector<const char*> deviceExtensions = {
		VK_KHR_SWAPCHAIN_EXTENSION_NAME,
		VK_KHR_BUFFER_DEVICE_ADDRESS_EXTENSION_NAME,
		VK_KHR_DEVICE_GROUP_EXTENSION_NAME,
		VK_KHR_DYNAMIC_RENDERING_EXTENSION_NAME // <- extension name macro for dyanmic rendering
	};
```

The most important thing to check is whether it is supported in your device before using it.
The below function is common steps to query device information in Vulkan.
- First, request extension count by enumerating properties
- second, enumerate properties as much as the extension count and fill out property vector
- Finally, While iterating available extension lists, you can compare it with required extension lists hardcoded in previous code snippets
```cpp
bool MKDevice::IsDeviceExtensionSupported(VkPhysicalDevice device)
{
	uint32 availableExtensionCount;
	vkEnumerateDeviceExtensionProperties(device, nullptr, &availableExtensionCount, nullptr);
	std::vector<VkExtensionProperties> availableExtensions(availableExtensionCount);
	vkEnumerateDeviceExtensionProperties(device, nullptr, &availableExtensionCount, availableExtensions.data());

	std::set<std::string> requiredExtensions(deviceExtensions.begin(), deviceExtensions.end());
	for (const auto& extension : availableExtensions)
	{
		requiredExtensions.erase(extension.extensionName); // erase extension name from set
	}

	return requiredExtensions.empty(); // if the vector is empty, every required extension is available.
}
```

Modern compute has multiple graphical processing units in a single system.
In my case, i have one in CPU as integrated unit and the other as discrete device.
If the multiple devices are suitable for your application requirements at the same time, you can rate them and pick one based on your taste. 
But i will skip the selection of device in this post.

Now we need to specify the features of extension in `pNext` chain.
The way of filling out required struct is very similar to previous implementation in `IsDeviceExtensionSupported` function.

In below code, `VkPhysicaDeviceFeatures2` is a main struct to specify all the extensions that i'm going to use in the application.
There are two things that you should remember before call `vkGetPhysicalDeviceFeatures2` call.

1. Initialize struct type (`.sType` field)
2. Connect additional features in `pNext` chain from device features struct to last extension features struct.
   Also, you should specify null pointer in the last struct so that vulkan API can notice it is the end point of chain.
```cpp
VkPhysicalDeviceFeatures2 deviceFeatures2{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2 };
VkPhysicalDeviceBufferDeviceAddressFeatures bufferDeviceAddressFeatures{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_BUFFER_DEVICE_ADDRESS_FEATURES };
VkPhysicalDeviceDynamicRenderingFeatures dynamicRenderingFeatures{ VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_DYNAMIC_RENDERING_FEATURES_KHR };

deviceFeatures2.pNext = &bufferDeviceAddressFeatures; 
bufferDeviceAddressFeatures.pNext = &dynamicRenderingFeatures;
dynamicRenderingFeatures.pNext = nullptr; // should mark as nullptr
```


# 3. Use Dynamic rendering in draw calls

Simply saying, There are four things to do when you render a scene with dynamic rendering extension.


## 3.1 Specify `VkPipelineRenderingCreateInfoKHR` struct and connect it into pNext chain of `VkPipelineCreateInfo`

- An example of how to specify `VkPipelineRenderingCreateInfoKHR` struct
```cpp
SetRenderingInfo(
	uint32 colorAttachmentCount, 
	VkFormat* pColorAttachmentFormats, 
	VkFormat depthAttachmentFormat, 
	VkFormat stencilAttachmentFormat
)
{
	renderingInfo.colorAttachmentCount = colorAttachmentCount;
	renderingInfo.pColorAttachmentFormats = pColorAttachmentFormats;
	renderingInfo.depthAttachmentFormat = depthAttachmentFormat;
	renderingInfo.stencilAttachmentFormat = stencilAttachmentFormat;
	renderingInfo.pNext = nullptr;
}
```

- An example of how to connect above struct into `pNext` chain
  Because we don't need render pass anymore, we remain it as null handle and assign above struct into `pNext` chain
```cpp
VkGraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.sType               = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
// other fields...

if (renderPass != nullptr)
{
	pipelineInfo.pNext      = nullptr;
	pipelineInfo.renderPass = *renderPass;
	pipelineInfo.subpass    = 0;
}
else
{
	pipelineInfo.pNext      = renderingInfo;
	pipelineInfo.renderPass = VK_NULL_HANDLE;
	pipelineInfo.subpass    = 0;
}
```


## 3.2 Specify Image attachment format per draw call

You may have several attachments (color attachments, depth and stencil attachment for example) in your single draw call.

With render pass, we just describe these attachment information as part of render pass.
But with dynamic rendering, we need to create simple attachment informations before rendering.

- An example of attachment info specification. 
```cpp
VkRenderingAttachmentInfoKHR colorAttachmentInfo = mk::vkinfo::GetRenderingAttachmentInfoKHR();
colorAttachmentInfo.imageView   = _vkOffscreenColorImageView;
colorAttachmentInfo.imageLayout = VK_IMAGE_LAYOUT_GENERAL;
colorAttachmentInfo.resolveMode = VK_RESOLVE_MODE_NONE;
colorAttachmentInfo.loadOp      = VK_ATTACHMENT_LOAD_OP_CLEAR;
colorAttachmentInfo.storeOp     = VK_ATTACHMENT_STORE_OP_STORE;
colorAttachmentInfo.clearValue = clearValues[0];

VkRenderingAttachmentInfoKHR depthAttachmentInfo = mk::vkinfo::GetRenderingAttachmentInfoKHR();
depthAttachmentInfo.imageView   = _vkOffscreenDepthImageView;
depthAttachmentInfo.imageLayout = VK_IMAGE_LAYOUT_DEPTH_ATTACHMENT_OPTIMAL;
depthAttachmentInfo.resolveMode = VK_RESOLVE_MODE_NONE;
depthAttachmentInfo.loadOp      = VK_ATTACHMENT_LOAD_OP_CLEAR;
depthAttachmentInfo.storeOp     = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachmentInfo.clearValue  = clearValues[1];
```

You may can notice a difference between dynamic rendering and render pass if you've ever written render pass code before.

It is absence of finalLayout in attachment format when you use dynamic rendering.
While a render pass do image layout transition implicitly based on initial and final layout, we need to do it ourselves explicitly during commands recording.

## 3.3 Specify `VkRenderingInfoKHR` to begin rendering

The below code shows the fields that we need to specify inside of rendering info struct.

```cpp
VkRenderingInfoKHR renderingInfo{};
renderingInfo.sType                = VK_STRUCTURE_TYPE_RENDERING_INFO_KHR;
renderingInfo.pNext                = VK_NULL_HANDLE;
renderingInfo.flags                = flags;
renderingInfo.renderArea           = renderArea;
renderingInfo.layerCount           = 0;
renderingInfo.viewMask             = 0;
renderingInfo.colorAttachmentCount = colorAttachmentCount;
renderingInfo.pColorAttachments    = pColorAttachments;
renderingInfo.pDepthAttachment     = VK_NULL_HANDLE;
renderingInfo.pStencilAttachment   = VK_NULL_HANDLE;
return renderingInfo;
```

# 3.4 Transition image layout (image memory pipeline barrier)

Let's see an example to see clearly how we need to transition image layout appropriately.

First of all, Vulkan is a platform-agnostic API. Hence, we need to get proxy addresses of `vkCmdBeginRenderingKHR` and `vkCmdEndRenderingKHR` functions. 
```cpp
auto vkCmdBeginRenderingKHR = (PFN_vkCmdBeginRenderingKHR)vkGetInstanceProcAddr(_mkInstance.GetVkInstance(), "vkCmdBeginRenderingKHR");
auto vkCmdEndRenderingKHR = (PFN_vkCmdEndRenderingKHR)vkGetInstanceProcAddr(_mkInstance.GetVkInstance(), "vkCmdEndRenderingKHR");
if (!vkCmdBeginRenderingKHR || !vkCmdEndRenderingKHR)
{
	throw std::runtime_error("Unable to dynamically load vkCmdBeginRenderingKHR and vkCmdEndRenderingKHR");
}
```

However, Depending on your API setup, you may be able to use `vkCmdBeginRendering` and `vkCmdEndRendering` without query proxy addresses.

Swapchain image's layout is undefined initially.
To write any data on the image, we need to transition undefined layout to color attachment optimal layout.
This `VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL` parameter helps a `TransitionImageLayout` function to choose proper memory access flags, which is `VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT` in this case.

we also need to transition initial undefined depth image layout to depth attachment optimal layout.

After we records every commands related to post-draw (in `DrawPostProcess`), we need to transition the previous layout (`VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL`) to `VK_IMAGE_LAYOUT_PRESENT_SRC_KHR`  so that we can present the image on screen.

You may wonder why i didn't transition depth attachment layout to present source layout. It's because i don't use it for presentation except for debugging purpose.

```cpp
mk::vk::TransitionImageLayout(
	commandBuffer,
	_mkSwapchain.GetSwapchainImage(swapchainImageIndex),
	_mkSwapchain.GetSwapchainImageFormat(),
	VK_IMAGE_LAYOUT_UNDEFINED,
	VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL,
	colorRange
);
mk::vk::TransitionImageLayout(
	commandBuffer,
	_mkSwapchain.GetDepthImage(),
	_mkSwapchain.GetDepthFormat(),
	VK_IMAGE_LAYOUT_UNDEFINED,
	VK_IMAGE_LAYOUT_DEPTH_ATTACHMENT_OPTIMAL,
	depthRange
);

// 
// swapchain color attachment format specifications ... 
//

// 
// swapchain depth attachment format specifications ... 
//

// postRenderInfo specifications...

// begin post rendering
vkCmdBeginRenderingKHR(commandBuffer, &postRenderInfo);
// draw post process
DrawPostProcess(commandBuffer, swapchainExtent);
// end post rendering
vkCmdEndRenderingKHR(commandBuffer);

mk::vk::TransitionImageLayout(
	commandBuffer,
	_mkSwapchain.GetSwapchainImage(swapchainImageIndex),
	_mkSwapchain.GetSwapchainImageFormat(),
	VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL,
	VK_IMAGE_LAYOUT_PRESENT_SRC_KHR,
	colorRange
);
```

# 4. Conclusion
---

That's all !
Although it requires explicit pipeline barrier and it is quite complex at the first time, It fit into Vulkan design principle ('explicit for everything') while removing all the unintuitive render pass and frame buffer setup.

The rendered result is exactly same as draw call with render pass.
![](images/Pasted%20image%2020240817204253.png)