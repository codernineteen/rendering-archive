
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