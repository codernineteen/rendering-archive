
# Mentality

1. progress != visible works.
	Although the little drawing a triangle looks too simple, it requires lots of neccessary knowledge about Vulkan.

2. human always forget.
	graphics pipeline in Vulkan is very deep. so keep pausing and reminding that where we are now and what we've learned.

3. There are many ways to do something.
4. Refer to Vulkan specification as soon as possible.

## Vulkan Mental model

- shader
```GLSL
#version 450

// Rather contrived shader for illustrative purposes

layout (location = 0) in vec3 position;

layout (location = 1) in vec2 uv;

layout (binding = 0, set = 3) uniform UBO
{
  mat4 projection;
} ubo[4];

layout (binding = 1, set = 1) uniform sampler2D noise_textures[3];

layout (binding = 2) buffer Buffer
{
  mat4 something;
} buffer;

layout (push_constant) uniform Camera {
  vec2 pos;
} camera;

layout (location = 0) out vec3 out_uv;

out gl_PerVertex
{
  vec4 gl_Position;
};

void main()
{
  // shader code
}

```


- memory
Vulkan have many types of memory.(device visible, coherent, local, etc)
Like we handle memory in c/c++ language, there are similar way to allocate/deallocate memory in using Vulkan. (RAII pattern becomes handy in this case.)


- synchronization
GPU -  separate thread of execution conceptually
Memory - shared memory (Don't write when it is being used.)

[Possible cases]
	- dependent commands handling -> using semaphore
	- memory ownership, layouts or visiblity -> memory barrier (ordering?)

the various graphics, transfer and compute queues can be thought of as separate threads within **GPU**.

In Vulkan, without help of driver, developer need to manage reference counting stuff.


# Read [Vulkan Tutorial](https://vulkan-tutorial.com/) or [API without Screats](https://www.intel.com/content/www/us/en/developer/articles/training/api-without-secrets-introduction-to-vulkan-part-1.html)

1. don't be afraid to copy-paste code in tutorial
   Is mindless typing code really helpful?
2. follow code convention in the tutorial
3. read the specification when we encounter new concepts in tutorial

# Do you own thing
Once we've done the tutorial step, we need to do our stuff, that is, making something!
Custom renderer can be a good start.
(p.s we can use this good open-source memory allocator here . [VulkanMemoryAllocator](https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator)
also it will be good practice to write custom allocator on our own)
