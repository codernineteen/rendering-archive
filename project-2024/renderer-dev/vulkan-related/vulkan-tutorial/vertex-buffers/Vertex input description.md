
We need to replace hard-coded vertex data with a vertex buffer objects in memory

a few ways to creating vertex buffers :
 - CPU visible buffer and memcpy to copy vertex data into it directly
 - a staging buffer to copy the vertex data to high performance memory

# Vertex shader
```GLSL
#version 450 
// vertex attributes position and color
layout(location = 0) in vec2 inPosition; 
layout(location = 1) in vec3 inColor; 

layout(location = 0) out vec3 fragColor; 

void main() { 
	gl_Position = vec4(inPosition, 0.0, 1.0); 
	fragColor = inColor; 
}
```

Note that some types like `dvec3` uses multiple slot
```GLSL
layout(location = 0) in dvec3 inPosition;
layout(location = 2) in vec3 inColor;
```

# Vertex Data
Specify vertex data using glm library.
```cpp
// Vertex data
struct Vertex {
    glm::vec2 pos;
    glm::vec3 color;
};

// interleaving vertex attributes
const std::vector<Vertex> vertices = {
    {{0.0f, -0.5f}, {1.0f,0.0f,0.0f}},
    {{0.5f, 0.5f}, {0.0f,1.0f,0.0f}},
    {{-0.5f, 0.5f}, {1.0f,0.0f,1.0f}},
}
```



# Binding descriptions
Tell Vulkan how to pass this data format to the vertex shader.
Two information strucures are required :
- `VkVertexInputBindingDescription`
	specifies the number of bytes between data entries and whether to move to the next data entry after each vertex or after each instance.
	Parameters :
	- binding : index of the binding in the array of bindings
	- stride : number of bytes from one entry to the next.
	- inputRate : 
		- `VK_VERTEX_INPUT_RATE_VERTEX`: Move to the next data entry after each vertex
		- `VK_VERTEX_INPUT_RATE_INSTANCE`: Move to the next data entry after each instance (for instance rendering)

# Attribute description
- `VkVertexInputAttributeDescription`
	Matched with number of vertex attributes
	Parameters :
	- binding - from which binding the per-vertex data comes
	- location - location directive of the input in the vertex shader
	- format - the type of data for the attribute
		 - `float`: `VK_FORMAT_R32_SFLOAT`
		- `vec2`: `VK_FORMAT_R32G32_SFLOAT`
		- `vec3`: `VK_FORMAT_R32G32B32_SFLOAT`
		- `vec4`: `VK_FORMAT_R32G32B32A32_SFLOAT`
		- `ivec2`: `VK_FORMAT_R32G32_SINT`, a 2-component vector of 32-bit signed integers
		- `uvec4`: `VK_FORMAT_R32G32B32A32_UINT`, a 4-component vector of 32-bit unsigned integers
    	- `double`: `VK_FORMAT_R64_SFLOAT`, a double-precision (64-bit) float
	- offset - specifies the number of bytes since the start of the per-vertex data to read from


# Pipeline vertex input
Set up graphics pipeline to accept vertex data in this format.

```cpp
VkPipelineVertexInputStateCreateInfo vertexInputInfo{};
auto bindingDescription = Vertex::getBindingDescription();
auto attributeDescriptions = Vertex::getAttributeDescriptions();
vertexInputInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertexInputInfo.vertexBindingDescriptionCount = 1;
vertexInputInfo.vertexAttributeDescriptionCount = static_cast<uint32_t>(attributeDescriptions.size());
vertexInputInfo.pVertexBindingDescriptions = &bindingDescription;
vertexInputInfo.pVertexAttributeDescriptions = attributeDescriptions.data();
```
