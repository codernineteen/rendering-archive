# Overview of graphics pipeline

![[../../../../../images/Pasted image 20240107164017.png]]

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

---

# Shader Modules

shader code in Vulkan has to be specified in a bytecode format. (Unlike GLSL, HLSL), which is called SPIR-V.

Advantage :
- GPU vendor compilers( turn shader code into native code ) are less complex
- it is straightforward

We don't need to write this bytecode by hand. There is a compiler that compiles GLSL to SPIR-V.

## GLSL

a shading language with a C-style syntax.
GLSL uses global variables to handl input and output.

# Vertex shader
- processes each incoming vertex
- input
	- vertex attributes(model space position, color, normal and texture coordinates)
- output
	- final position in clip coordinates
		clip coordinate is '4D vector' that is subsequently turned into a normalized device coordinate by dividing the whole vector by its last component('w' in general)
	- attributes for fragment shader
![[../../../../../images/Pasted image 20240107212339.png]]

There is a built in variable named 'gl_Position' as the output in vertex shader

# Fragment shader
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


# Per-vertex color
- set color attributes in vertex shader
- pass per fragColor to fragment shader
- use it in fragment shader.

# Compile the shaders
GLSL don't have official extension.
we simply use '.vert' for vertex shader and '.frag' for fragment shader.

1. Put the shaders in 'shaders' directory together
2. Write batch script and use glslc binary from Vulkan SDK 
```bat
C:/VulkanSDK/x.x.x.x/Bin/glslc.exe shader.vert -o vert.spv C:/VulkanSDK/x.x.x.x/Bin/glslc.exe shader.frag -o frag.spv 
pause
```

# Loading a shader