********
# Wave Interference
When two or more waves are overlapped in the same region, we call it as 'wave interference'.
![](../../images/Pasted%20image%2020240806165907.png)

Regardless of directions of each waves, the height of total waves will be determined by each wave's contribution.
![](../../images/Pasted%20image%2020240806170242.png)

If the waves are overlapped partially in the region, we can represent the total wave in sub divisions as below.
![](../../images/Pasted%20image%2020240806170424.png)

## constructive interference
 When the phases of two waves are exactly located in same region, they contstruct a wave that is twice as big as the original wave. Then we call it 'constructive interference', more precisely 'totally constructive interference' in this case.
Even though the waves aren't lined up exactly same region, they still construct bigger waves and we can call it constructive interference.
![](../../images/Pasted%20image%2020240806171006.png)

## destructive interference
Let's take a case that one of the waves is shifted left as much as half of its frequency.
In this case, the overlapping two waves generates a wave whose height converges to zero in all possible points.
we call it as 'totally destructive interference.'
![](../../images/Pasted%20image%2020240806171425.png)

---
# Media

- homogeneous medium : a volume filled with uniformly spaced identical molecules. **Liquids and non-crystalline solids can be optically homogeneous if their composition is pure**
  
When a light wave pass through such a media, scattered waves are lined up and interfere destructively in all directions except for the direction of original wave.

Only the phase velocity and amplitude are changed as a final result.

We can represent the ratio of phase velocities of original wave and new wave as '*index of rafraction* (IOR)', denoted by '$n$'

Some media is absorptive, so they convert part of light energy to heat.
This causes the amplitude of the wave to decrease exponentially with distance.
The rate of decrease is defined by '*atteunation index*', denoted by '$k$'


# Surfcaces
In the perspective of 'optics', an object surface can be divided into two parts, which are volumes with different IOR.
we assume outer volume as air with 1.003 IOR and inner volume is totally dependent on the substance from which the object is made. 

There are two sides of a surface.
The one is 'the effect of the substances' and the other is 'geometry'.

## Substance side view
In related to substances with perfectly planar surface, when there is a different between outside IOR and inside IOR, this IOR discontinuity leads light to scatter in a specific way. (the projection of electric filed vector to the surface plane mut match on the either side of the surface.)
Here are several implications :
- The scattered waves must be either in same phase(refraction) or 180 degree out of phase(reflectance)
- The scattered waves must have the same frequency as the incident wave. If it is a combined waves, we can decompose it into several monochromatic components and apply same principle.
- The phase velocity will be changed depending on the inner IOR when a light moves from one medium to another. The velocity changes proportionally to the ratio of two IOR ($n_1/n_2$)
	-> The transmittance angle is defined by *snell's law*.


## geometry side view

Actually, it is impossible for a surface to be perfectly planar. Every surface has irregularities in real world.

Irregularities with a size in the range of 1~100 wavelenghts cause the surface to behave differently than a flat plane (diffraction.)

However, we use geometrical optics typically in rendering (ray tracing is also type of geometrical optics if it doesn't trace polarization feature of light.)

When irregularities on surface are too small to be individually rendered, we refer to them as 'microgeometry'.
This microgemetry feature changes normal at different points on the same surface, thus changing the reflected and refracted light directions.

In Rendering, Instead of modeling an 3d asset with microgeomtry explicitly, we takes an statistical approach to describe the effect.


# Subsurface scattering
A refracted ray may be scattered inside of the object and we call this as 'subsurface scattering'.
When Shading scale unit is bigger than enter-exit distance, local diffuse shading model can replace the effect.
On the other hand, If the enter-exit distance is longer than the shading scale, we may need to apply global subsurface scattering technique to represent scene more accurate.
![](../../images/Pasted%20image%2020240809163128.png)


---

# Camera

 In physical camera, there is a sensor to detect the irradiance value over its surface, which is called 'photodiodes'.
However irradiance sensors cannot produce an image themselves.
 To generate an image, we need a full imaging system including a small 'aperture'.
A lens placed at aperture focuses incoming light so that each senser receives light from only small set of incoming directions.

These sensors measure average radiance which quantifies the brightness and color of a single ray of light rather than average irradiance.

**Physical Pinhole camera**
![](../../images/Pasted%20image%2020240809165357.png)

**Typical rendering system model**(imitate pinhole camera system)
![](../../images/Pasted%20image%2020240809165438.png)

**Physicall correct camera with a lens**
![](../../images/Pasted%20image%2020240809165509.png)


---
# BRDF

- Physically based rendering $\approx$ computing the radiance entering the camera along some set of view rays.
  The goal is to compute $L_i(c,-v)$, where $c$ is the camera position and $-v$ is the direction along the view ray. (Here we use $-v$ following convention because view ray direction always points toward camera.)

In a simple scene, we don't care about '*participating media*'(which participates in the light transport).
Therefore, the radiance entering the camera is equal to the radiance leaving the closest object surface in the direction of the camera.
$$L_i(c,-v) =L_o(p,v)$$
, where $p$ is the closest hit point of the view ray on certain object.

In real world, The goal may be to compute $L_i$. However, it is to compute $L_o$ because we generate rays from camera.


To make the topics simple, we can focus on only reflectance for now.
Local reflectance is quantified by the *bidirectional reflectance distribution function*, shortly BRDF.

The BRDF is denoted as $f(l,v)$ and it takes two parameters which are incoming light direction and outgoing view direction.

Each directions has two degrees of freedom. (one degree for horizontal plane, the other degree for surface normal).
![](../../images/Pasted%20image%2020240809171317.png)
By the way, in the case of isotropic material, we can only represent BRDF with three parameters because such BRDF remains same when incoming and outgoing direction are rotated around surface normal.


In real-time rendering, we returns a spectrally distributed value as an output of BRDF (RGB triples).
- reflectance equation to compute outgoing direction.
$$L_o(p,v) = \int_{l\in\Omega}f(l,v)L_i(p,l)(n\cdot l)dl$$
	- $l\in\Omega$ : integration is performed over $l$ vectors that lie in the unit hemisphere above surface. (any incoming directions including both direct and indirect lighting effects)
	- $dl$ : differential solid angle around vector $l$

By parameterizing above reflectance equation, we can represent the equation in nested integral form.
$$L_o=\int_{\phi_i=0}^{2\pi}\int_{\theta_i=0}^{\pi/2}f\cdot L_i\cdot cos\theta_i\cdot \sin{\theta_i}d\theta_id\phi_i$$, where $\sin{\theta_i}d\theta_id\phi_i$ is parameterized differential solid angle and $cos\theta_i$ is $(n\cdot l)$ by cosine law

---
# Reference

- [Khan academy](https://www.khanacademy.org/science/physics/mechanical-waves-and-sound/standing-waves/v/wave-interference-pulses)
- [Real-time Rendering 4th ed.](https://www.realtimerendering.com/)