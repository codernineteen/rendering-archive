

# Shader definition

- shader : functions that a GPU device can run
	In terms of path tracing application, we write functions for each pixel in an image.


# Shader type

- compute shader (for computer pipeline)
	Compute shader is designed for general-purpose computations.
	compute shader can use ray queries to perform ray tracing operation, which is originally possible to be done in ray tracing pipeline.
	
- Ray generation, miss, closest hit, callable and intersection shader
	These shaders is for ray tracing pipeline.
	They can be accessed by shader binding table which is created at the moment of creation of ray tracing pipeline.
	- ray generation shader - generate rays by defining a ray origin and direction for each pixels and trace generated rays.
	- ray miss shader - returns a sky color (default color) against the rays that are not intersected with any other meshes.
	- closest hit shader - takes a ray-mesh intersection and return the properties of a mesh at the intersection point.

- vertex, tessellation(control and evaluation), geometry, fragment , mesh shaders
	These shaders are for traditional graphics pipeline.
	An application with graphics pipeline use these shaders for some tasks like vertex processing, rasterization and coloring pixels.


One of the important things regard to pipelines is that we can combine several pipelines into a single renderer application.
For example, we can use compute shader to compute ambient occlusion while using a ray tracing pipeline for reflections or global illumination.


# Work group in compute shader

- GPU invokes the entry point of compute shader for every point in a 3D grid.
- application defines how to divide this grid into **3D work groups**

Compute shader examples below : 
- Multiply 15 by 13 matrix by scalar of 2
- compose each work group 16 x 8 x 1 size
- Then it requires a grid of 1 x 2 x 1 work groups.


# How shader compilation works

- In CPU,  we must convert high-level language code (such as C/C++) into machine level codes by following the instruction set supported by current CPU architecture.
  Someone might compile a c++ program using Clang (c++ -> LLVM's intermediate representation) to optimizes the code.
- GPU have a much wider variety of architectures and instructions sets.i
	Vulkan takes a similar approach as LLVM. (glslangValidator translates GLSL into SPIR-V , then SPIR-V -> GPU assembly) 