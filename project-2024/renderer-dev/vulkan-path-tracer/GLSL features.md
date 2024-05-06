
- version specifier
	`#version <version_num>`
- add extension
	`#extension <name> : <option>`

- layout qualifiers 
	`layout(binding = <binding_num>, set = <set_num>) <storage qualifiers>`

- storage qualifiers  are keywords such as `buffer`, `uniform`
		ex) `tlas` has the `uniform` storage qualifier. 
		Shader invocations can read and write it, but the graphics API can't access it.

	In GLSL, every variable either has or does not have a storage qualifer.
	For example, 
	 -  a `vec3 rayOrigin` variable doesn't have any storage qualifer.
	 - `layout(binding = 1, set = 0) uniform accelerationStructureEXT tlas` has `uniform` storage qualifer.

	- `buffer` : stored in a buffer object, can be read and written
	- `in` : comes from another pipeline stage (ex. input data from vertex processing stage to fragment coloring stage)
	- `out` : ouput from current stage to another pipeline stages.

	All Variables related to Vulkan's descriptors will have `buffer` or `uniform` storage qualifers.

Here are more examples from `vk_mini_path_tracer` tutorial.
- uniform buffer (read only)
```GLSL
layout(binding = 2, set = 0, scalar) uniform nameOfBlockHere
{
  vec4   backgroundColor;
  mat4x4 viewMatrix;
  float  focusDistance;
  vec3   lightPositions[16];
  vec3   lightColors[16];
};
```

- block of push constant (potentially faster uniform buffer)
```GLSL
layout(push_constant) uniform nameOfOtherBlockHere
{
  uvec2 imageResolution;
}
```