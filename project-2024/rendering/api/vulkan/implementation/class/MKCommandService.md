MKComandService is accessible globally.
The instance of this class in constructed at the beginning of program runtime and it is initialized at Device creation time.

The service provides several things :
- abstractions for verbose command recording
- easy-to-use single time command with lambda function.
- submit recorded commands to queue 
# API

- Init command service
- Submit command buffer to queue
- reset command buffer for reusing long-living command buffer.
- Execute single time commands
  example ) 
  ```cpp
  GCommandService->ExecuteSingleTimeCommands([&](VkCommandBuffer commandBuffer) { // getting reference parameter outer-scope
	VkBufferCopy copyRegion{};
	copyRegion.size = size;
	vkCmdCopyBuffer(commandBuffer, src, dest, 1, &copyRegion);
	});
  ```

# User

- MKGraphicsPipeline
	1. record 
	2. begin/end single time commands 
	3. submit buffer to queue 
	4. reset

- MKDevice
	1. Initialize the GCommandService instance