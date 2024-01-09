# Overview of graphics pipeline
---
![](../../../../../../images/Pasted%20image%2020240107164017.png)

- input assembler : collects the raw vertex data for the **buffers**.
- vertex shader : 
	- runs for every vertex. 
	- applies transformations from model space to screen space in general.
	- passes per vertex data down the pipeline
- tessellation shader:
	- subdivide geometry -> increase mesh quality
- geometry shader:
	- run on every primitive(triangle, line, point)
	- not used much in these days
- rasterization :
	- discretizes the primitives into fragments == pixel elements on framebuffer
	- depth testing
- fragment shader :
	- for every fragments that survive in framebuffer
	- color, depth value 
	- can interpolate data from vertex shader
- color blending :
	- mix different fragments that map to the same pixel in the frame buffer, which is called blending
	- ex. transparency
	- alpha blending

Stages with green color - fixed-function stages 
Stages with yellow color - programmable stages

In Vulkan, Graphics pipeline is almost immutable. Most of changes in pipeline requires recreation of the pipeline from scratch.

programmable stages are optional
-> this is useful for shadow map generation (create depth values first with disabling unneccessary stages)

<div style="height: 2px; background-color: greenyellow;"></div>

# Shader Modules
---

shader code in Vulkan has to be specified in a bytecode format. (Unlike GLSL, HLSL), which is called SPIR-V.

Advantage :
- GPU vendor compilers( turn shader code into native code ) are less complex
- it is straightforward

We don't need to write this bytecode by hand. There is a compiler that compiles GLSL to SPIR-V.

## GLSL
---
a shading language with a C-style syntax.
GLSL uses global variables to handl input and output.

## Vertex shader
---
- processes each incoming vertex
- input
	- vertex attributes(model space position, color, normal and texture coordinates)
- output
	- final position in clip coordinates
		clip coordinate is '4D vector' that is subsequently turned into a normalized device coordinate by dividing the whole vector by its last component('w' in general)
	- attributes for fragment shader
	- 
![](../../../../../../images/Pasted%20image%2020240107212339.png)
There is a built in variable named 'gl_Position' as the output in vertex shader

## Fragment shader
- produce  a color and depth for the 'framebuffer'.
- we have to specify our own output variable for each framebuffer where the `layout(location=0)` modifier specifies the index of the framebuffer. (Unlike vertex shader's 'gl_Position', there is no built-in output variable in fragment shader)

Example code
```GLSL
#version 450 

layout(location = 0) out vec4 outColor; 

void main() { 
	outColor = vec4(1.0, 0.0, 0.0, 1.0); 
}
```


## Per-vertex color
---
- set color attributes in vertex shader
- pass per fragColor to fragment shader
- use it in fragment shader.

## Compile the shaders
---
GLSL don't have official extension.
we simply use '.vert' for vertex shader and '.frag' for fragment shader.

1. Put the shaders in 'shaders' directory together
2. Write batch script and use glslc binary from Vulkan SDK 
```bat
C:/VulkanSDK/x.x.x.x/Bin/glslc.exe shader.vert -o vert.spv C:/VulkanSDK/x.x.x.x/Bin/glslc.exe shader.frag -o frag.spv 
pause
```

## Loading a shader
---
Using standard file stream library, we can load, read and write the file into buffer.
- std::ios::ate - read at the end of the file
	One of advantage to use ios::ate is that we can utilize the file size to create a buffer.
- std::ios::binary - binary format without text transformation
```cpp
#include <fstream>
.
.
.
static std::vector<char> readFile(const std::string& filename) { 
	std::ifstream file(filename, std::ios::ate | std::ios::binary); 
		if (!file.is_open()) { 
			throw std::runtime_error("failed to open file!"); 
		} 
}
```

## Creating a shader modules
---
We have to wrap shaders in a `VkShaderModule`
Shader Modules are thin wrapper around the shader bytecode

1. specify pointer to the buffer with the bytecode and length of it
2. cast char pointer to uint32_t pointer (reinterpret_cast)
		alignment requirement should be satisfied when using `reinterpret_cast`.

Shader compilation and linking of SPIR-V doesn't happen until the graphics pipeline created.
So we can just define the shader modules as local variable and destroy **shader code** synchronously after the creation of **modules**.

## Shader stage creation
---
To use shaders, we need to assign them to a *specific pipeline stage* through `VkPipelineShaderStageCreateInfo`

- sType : a macro about this struct type
- stage : a stage in pipeline the shader is going to be used
- module :  a shader module struct we created previously
- pName : function name as entrypoint
- pSpecializationInfo(optional) : vales for shader constants.
	- With this field, we can even use single shader module where its behavior is configured at pipeline creation by specifying different values for the constants used in it.




<div style="height: 2px; background-color: greenyellow;"></div>


# Fixed functions
---
In Vulkan, we have to be explicit about most pipeline states.

## Dynamic state
---
- a limited amount of states are dynamic at draw time.
	- size of viewport
	- line width, blend constants
we need to specify create info for dynamic states if we want to use those.


## Vertex input
---
- `VkPipelineVertexInputStateCreateInfo` - formate of the vertex data
	1. bindings : spacing between data and whether the data is per-vertex or per-instance.
	2. attribute descriptions : types of the attributes passed to input of the vertex shader (offset required)

**struct**
- sType
- vertexBindingDescriptionCount
- vertexAttributeDescriptionCount
- pVertexBindingDescriptions - members point to an array of structs that describe details for loading vertex data
- pVertexAttributeDescriptions 

## Input assembly
---
- `VkPipelineInputAssemblyStateCreateInfo` struct describes :
	1. what kind of geometry will be drawn from the vertices <- define it in 'topology' member.
	- `VK_PRIMITIVE_TOPOLOGY_POINT_LIST`: points from vertices
	- `VK_PRIMITIVE_TOPOLOGY_LINE_LIST`: line from every 2 vertices without reuse
	- `VK_PRIMITIVE_TOPOLOGY_LINE_STRIP`: the end vertex of every line is used as start vertex for the next line
	- `VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST`: triangle from every 3 vertices without reuse
	- `VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP `: the second and third vertex of every triangle are used as first two vertices of the next triangle
	2. whether primitive restart should be enabled(?)

vertex buffer loaded in sequential order in general, but we can specify indices in element buffer to reuse vertices. (set primitiveRestartEnable VK_TRUE)

## Viewports and scissors
---
viewport : a region of the framebuffer that the output will be rendered to 
	- width and height : we are goint to stick to swap chain's image size related to 'width' and 'height'
	- min/max depth : range of depth
scissor : any pixels outside of scissor rectangle will be discarded by the rasterizer
![](../../../../../images/Pasted%20image%2020240108152023.png)
To render entire framebuffer, just set it to swapChainExtent we specified before.

However, rather than define states of viewport and scissors statically, it will be much more flexible to set it as dynamic states as we've already seen before.



## Rasterizer
--- 
vertices -> vertex shader(shape) -> geometry -> **Rasterizer** generates fragments -> fragment shader

Rasterizer also can do detph testing and face culling
- `VkPipelineRasterizationStateCreateInfo`
	- sType
	- depthClampEnable : If VK_TRUE set, cull an object outside of near and far planes.
	- rasterizerDiscardEnable : geometry never passes through the rasterizer. Hence no output to framebuffer
	- polygonMode :  determines how fragments are generated for geometry
		- `VK_POLYGON_MODE_FILL`: fill the area of the polygon with fragments
		- `VK_POLYGON_MODE_LINE`: polygon edges are drawn as lines
		- `VK_POLYGON_MODE_POINT`: polygon vertices are drawn as points
	- lineWidth : tickness of line
	- cullMode : type of culling to use
		- `VK_CULL_MODE_NONE` specifies that no triangles are discarded
		- `VK_CULL_MODE_FRONT_BIT` specifies that front-facing triangles are discarded
		- `VK_CULL_MODE_BACK_BIT` specifies that back-facing triangles are discarded
		- `VK_CULL_MODE_FRONT_AND_BACK` specifies that all triangles are discarded.
	- frontFace : specify vertex ordering (clockwise or counter-clockwise)
	- depthBiasXXX : rasterizer can alter depth values by adding a constant or biasing them based on fragment's slope


## Multisampling
multisampling is one of the ways to perform anti-aliasing.
-> combining fragment shader results of multiple polygons to the same pixel.
## Depth and stencil testing
If we use a depth or stencil buffer (for example, shadow volume), we need to specify a create info for this either
## Color blending
---
fragments generated by rasterizer -> fragment shader -> a color for a pixel 
This returned color needs to be combined with the color that is already in the framebuffer, which is called color blending.

Two ways to blend colors : 
1. Mix the old and new value to produce final one (used for alpha blending)
2. combine the old and new value using bitwise operation

Two structs to configure color blending
- `VkPipelineColorBlendAttachments` : configuration per attached framebuffer 
- `VkPipelineColorBlendStateCreateInfo` : contains the global color blending settings.

Refer to [Blending factor and operations])(https://registry.khronos.org/vulkan/specs/1.3-extensions/html/chap30.html#VkBlendFactor) for more informations
## Pipeline layout
- uniform variables in shaders - globals similar to dynamic state that can be changed at a drawing time.
- ex) transformation matrix, texture samplers

These uniform variables need to be specified during pipeline creation by creating a `VkPipelineLayout` object
This struct provides 'push constants' field that are another way of passing dynamic values to shader.