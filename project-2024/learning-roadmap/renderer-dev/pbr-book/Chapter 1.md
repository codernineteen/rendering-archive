
# Photorealistic rendering and the ray tracing algorithm


Ligth is a wave-like manifestation in the electromagnetism framework.

motion of electrically charged particles produces a disturbance of a surrounding electric field that propagtes away from the source.

electric oscillation -> causes secondary oscillation of the magnetic field -> reinforces an oscillation of electric field again.
This interaction between two fields leads to a self-propagating wave that can travel extremely large distance. (ex. distant stars visible in the sky).

At the microscopic level, elementary properties like energy and momentum are **quantized**. -> the energy and momentum can only exist as 'quantum' (also called as photon)

In conclusion, Light have the side of both of particle-like behavior and wave-like nature.

---

The great fortune is that complex wave-like behavior of light is not important when simulating objects at the scale of **centimeters and meters**.
This fact leads to a more efficient computational approach 'ray tracing'

Key features of ray-tracing algorithm
- cameras : determine how and from where a scene is being viewed.
- ray-object intersections : we should detect whether a ray is intersected with certain objects or not. (typically returning the closest intersection along the way)
- Light source : a ray tracer model the distribution of light throughout the space.
- Visibility : determine wheter the path of a ray is not interrupted.
- scattering : Depending on a material property of an object, how a ray interacts with objects can be different.
- Indirect light
- Ray propagation : In related to atmosphere charateristic , determine how light energy remains in the environment.

---

# Camera and films

- a pinhole camera
![[Pasted image 20240103193925.png]]

The most important function of the camera is to define the portion of the scene that will be recorded onto the film.
Objects that are not in viewing volume cannot be imaged onto film.

- another way to think about pinhole cam
This is not practical way to build a real camera, but convenient abstraction for simulating objects.
In below image, a pinhole is referred to as the eye.
We are interested in  the amount of light traveling from the image point on film plane to the eye.
![[Pasted image 20240103194214.png]]


---

# Ray - object intersections

we can represent a ray in parametric form :
$$r(t)=o+td$$
where 'o' is the origin of ray and d is direction vector.
A point on ray can be achieved by varing the parameter 't'.

Let's think about ray intersection with sphere.
A sphere also can be represented as parametric form :
$$x^2+y^2+z^2-r^2=0$$
By substituting x,y and z with each components of ray(x,y,z component of ray), we can solve a equation to get intersection points.
- If there is no root, ray misses the spherer
- If there is roots, the smallest positive one is the intersection point.

But we still need more informations such as :
- surface properties at a certain point (material)
- additional geometric information about the intersection point (ex. surface normal, partial derivatives of position, etc)


Finally, speed of computing ray tracing is important topic.
In related subject, there are some optimizations like culling by removing unneccessary part while rendering a scene.

---
# Light distribution

Ray-object intersecion gives us :
-  a point to be shaded
- some information about the local geometry at the point

Our goal :
- find the amount of the light leaving this point in the direction of camera.

What we need to know :
- how much light is arriving at this point.

Here let's use point light to simplify this topic (In real world, it is area light-source in general).

we often want to know the amount of light power being deposited on the differential area surrounding intersection point P.

- Assumption : point light source has light power $\phi$ and it radiates light equally in all directions

Then, The power per area on a unit sphere surrounding the light is $\phi/(4\pi)$
It is obvious that the power per area at a point on the larger sphere must be less than the smaller one. (In below images, r2 is distance between origin and larger sphere boundary).
Hence, the power per area is proportional to $\frac{1}{r^2}$, which is called distance atteunation

![[Pasted image 20240105141101.png]]


Also, the amount of power deposited on differential area dA is proportional to $cos\theta$ if the dA is tilted by an angle $\theta$ away from the vector from 'p' to light source
![[Pasted image 20240105140916.png]]

Finally, the differential irradiance will be 
$$dE=\frac{\phi cos\theta}{4\pi r^2}$$

A scene with multiple lights are easily handled because illumination is linear.
In other words, the contribution of each light can be computed independently, then summed to obtain overall contribution. 
-> can apply true parallelism.

---

# Visibility

**Each light contributes illumination to the point being shaded only if the path from point to the light's position is unobstructed**
-> Visibility check is required

In below image, right light source doesn't illuminate point p.
![[Pasted image 20240105142454.png]]

In ray tracing, it is easy to determine whether the light is visible from the point by constructing a new ray whose origin is at surface point, which is called *shadow rays*.


At this point, we can compute vital informations
- a point location (by ray objects-intersection)
- the incident lighting (by differential irradiance and visibility check)

---

# Light scattering

Now we need to determine how the incident lighting is scattered at the surface.
Especially, the amount of light energy scattered back to camera position.

The amount of light scattered toward the camera is given by the product of **incident light energy** and **the BRDF function**
![[Pasted image 20240105143306.png]]

Each object in scene has material property, which is given by Bidirectional Reflectance Distribution Function(BRDF).
BRDF tells us how much energy is reflected from an incoming direction $\omega_i$ to an outgoing direction $\omega_o$.
$$f_r(p, \omega_o, \omega_i)$$
, where $p$ is a point on he surface.


---

# Indirect Light Transport

 a ray -> hit shiny object like mirror -> reflect the ray against the surface normal at the intersection point -> *recursively* invoke ray tracing routine.

 Details will be handled in secion 4 and 13

---
# Ray Propagation

In Vacuum environment, the light's power is distributed equally on the surface without decreasing along the way.
However, The presence of participating media such as smoke, fog, or dust can  invalidate this.

There are two ways in which a participating medium can affect the light propagating along a ray.
1. medium extinguish light
	   By absorbing or scattering it int a different location. (Compute Transmittance)
2. medium add to the light along a ray.
	this happens when the medium emits light like flame. (volume light transport equation)
