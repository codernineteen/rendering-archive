- To accelerate an algorithm to compute ray-mesh intersection, we need a bounding volume hierachy that reduces the number of intersection checking
- Vulkan provids us two interface to access this hierachy with bottom-level acceleration structure and top-level acceleration structure.![](../../../../images/Pasted%20image%2020240506214336.png)


## BLAS
- acceleration structures of triangles.

Required specifications for BLAS :
- location of vertex and index buffers on the GPU
	- we can provide this information by using buffer device address extension
- memory layout of the vertices. (stride and offset)
	- this depends on the `Vertex` structu that we define in application
	- a `Vertex` may hold a set of informations : normal vector, texture coordinates, positions(neccessary) and so on.
- format of indices
- number of vertices
- geometry data
- number of triangles
- start and end of triangles
- performance priority flag
- compacition flag if required
- updatable flag if required

## TLAS
- acceleration structures of instances
	an instance point to a BLAS and include a transform (represented as 3 x 4 affine matrix)

Specs :
- acceleration structure reference 
	device address of BLAS
- transform 
	3 x 4 transformation matrix. (can apply translate, scale, rotate, skew, etc)
- instance custom index 
	arbitrary 24-bit number that can be queried when there is intersection between ray and BLAS. (can get this using `rayQueryGetIntersectionInstanceCustomIndexEXT`)
- instance shader binding table record offset
	designed to store an offset for looking up material shaders
- `VkGeometryInstanceFlagsKHR` 
	specify how rays intersect with the instace during ray traversal.
- `mask`
	8-bit bitmask. 
	programatically select a set of instances to trace.