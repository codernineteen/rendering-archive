A common 3d model file format is 'obj'.
After we load the model using a [tiny obj loader](https://github.com/tinyobjloader/tinyobjloader/blob/release/tiny_obj_loader.h) library, we can convert it into vertices and indices.
Be sure to set uint32_t type for indices because a model has a lot more vertices than 65535 in general.

An OBJ file consists fo positions, normals, texture coordinates and faces
- faces  : consist of an arbitrary amount of vertices.
	- each face consists of an array of vertices
		- each vertex contains the **indices** of position, normal and texture coordinate attributes.
- `attrib` : it holds all of the positions, normals and texture coordinates
- `shapes` : all of the separate objects and their faces

LoadObj function in tiny obj loader supports automatic triangulation.