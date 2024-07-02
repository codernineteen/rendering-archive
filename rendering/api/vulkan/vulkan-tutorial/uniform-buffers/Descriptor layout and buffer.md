To move on 3d graphics, we need concept of spaces (model , view and projection space in general).
The matrices of these spaces is requred to be shared between shaders, thus we need to set it as global.
What is global in shader?  it is uniform variable!

- A resource description
	A way for shaders to freely access resources like buffers and images.

How to use :
1. specify a descriptor set layout during pipeline creation
	- descriptor set layout : the types of resource that are going to be accessed by the pipeline
	- descriptor set : actual buffer or image resources
1. allocate a descriptor set from a descriptor pool
2. Bind the descriptor set during rendering

# Descriptor set layout

To bind descriptor set layout, we use a struct `VkDescriptorSetLayoutBinding`
`VkDescriptorSetLayoutBinding` :
- binding :  the binding number used in the shader (ex. `layout(binding = 0)` in shader)
- descriptorType : type of descriptor
- descriptorCount : the number of values in an array of uniform buffer objects.
	- ex) a transformation for each of the bones in a skeleton for skeletal animation.
- stateFlags : specify in which shader stages the descriptor is going to be referenced.
- pImmutableSamplers : only texture sampler related

Then, use `VkDescriptorSetLayoutCreateInfo` to create a descriptor set by pointing to its layout binding.

# Uniform buffer
Create uniform buffer first before specifying the buffer that contains the UBO data for the shader. 

We're not going to use staging buffer here because we copy new data to uniform buffer every frame and it can cause unneccessary overhead.

Also, Multiple frames may be in flight at the same time. we need to have as many uniform buffers as frames in flight.

We need these fields in class to match number of uniform buffers to MAX_FRAMES_IN_FLIGHT.
- `std::vector<VkBuffer>`
- `std::vector<DeviceMemory>`
- `std::vector<void*>`

Because the buffers stay mapped to each of its void pointer for the whole application's lifetime (persistent mapping), we are not going to unmap the memory like vertex buffer creation.


# Update uniform data
We need to add two more headers which are :
- glm/gtc/matrix_transform.hpp : various matrix transformation support (rotation, projection, lookAt and so on)
- std::chrono : precise timekeeping for exact transformations regardless of framerate.

After adding these headers, we need to do
- update uniform buffer object every frame
	For example,
	glm::rotate for model rotating transformation
	glm::lookAt for view space
	glm::perspective for perspective projection


# Further discussion
Uniform buffer object is not the most efficient way to pass frequently changing values to the shader.
The more efficient way to pass small buffers of data to shaders are *push constants*
