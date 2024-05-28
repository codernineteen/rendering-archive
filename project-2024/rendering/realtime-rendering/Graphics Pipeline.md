
- The **locations and shapes** of the objects in the image are determined by their geometry
- **The appearance of the objects** is affected by material properties, light sources, textures and shading equations

# 1. Architecture
---

- a graphics pipelines are composed of several stages.
	The stages execute parallel and each stages are dependent on result of previous stage.
	- bottleneck : The slowest stage in a pipeline. The execution speed of the pipeline is dependent on this stage.

- The core structure of a graphics pipeline is constructed by Four main stages.
	1. Applictation
		Examples of type of task running on application :
		- collision detection, global acceleration algorithms
	2. Geometry processing
		- what it is to be drawn
		- how it should be drawn
		- where it should be drawn
	3. rasterization
		- Take three vertices -> forming a triangle -> find all pixels related to the triangle.
	4. pixel processing
		- define per-pixel operations.
	Each of above stages is usually a 'small pipeline' itself. The main stages consists of several sub-stages.

- Rendering speed is notated as 'frame per second' or '*Hertz*'(1 / seconds) 

## Application stage

- developer has full control of implementaions of app.
- The details of implementation in application side can affect the performance of GPU operations.
	- ex) some of application works can be performed by 'compute shader', which is general purpose shader program and treated as 'highly parallel general process' by app.
- Finally, *rendering primitives* are passed into geometry processing stage.

## Geometry Processing
- In this stage, GPU handles per-triangle and per-vertex operations.
	(stages of pipeline may be different according to API)
	![](../../../images/Pasted%20image%2020240515224406.png)


- Vertex shading
	==Two main tasks of vertex shading :==
	- compute position for a vertex 
		do operations for transformations from model space to projection space.
	- evaluate whatever the programmer may like to have as vertex output (it could be normal or texture coordinates for example)

	==Vertex transformation :==
	- originally a model resides in 'model space'.
	- Then we apply transform onto the model.
	- It is the vertices and normals of the model that are transformed by the trasformation matrices. (Note that a single model is actually composed of a set of indices, vertices, normals and so on.)
	- After the transformations are applied to model, it moves from Model space to 'world space'.
	- One of the important things is that only the models that the camera sees are rendered. (The camera itself has a location in world space and it points to somewhere toward scene by its direction.)
	- To make **projection** and **clipping** easier, the camera and all the models are transformed with *view transform*
		- What is 'view transform' for?
			Its purpose is to place the camera at the origin and aim it while making it look in the direction of negative z-axis, y-axis point upward and x-axis point to right
			![](../../../images/Pasted%20image%2020240517110331.png)
	
	- Note) To make a scene look realistic, It is not enought just to render the shape and position of objects. At the moment, we need 'shading' which involves 'shading equation' at various points on the object.
	
	**Optional stages :**
	- tessellation : make the object represent detail by making it contain more traingles.
	- geometry shader : it is similar to tessallation, but it can produce new vertices. (this shader is commonly used on particle generation)
---

primitives -> rasterizer -> visible pixel -> screen displas the contents of the color buffer.
To avoid seeing rasterization steps, we use double buffering. (offscreen rendering)
Once a scene has been rendered in the back buffer , it is swapped with the contents with the front buffer.
(For reference, we use in-flight images to avoid tearing on screen in Vulkan.)




# References

- Real-time Rendering 4th.ed