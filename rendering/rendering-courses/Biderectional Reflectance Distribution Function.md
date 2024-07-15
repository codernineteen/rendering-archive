
# Material model

- specular reflection : one possible outgoing direction against incoming light
- diffuse reflection : many possible outcomes, many possible directions
	can generate many combinations and one of the combinations can be obtained by smart random sampling.
- spread : mixture of specular and diffuse
![](../../../../images/Pasted%20image%2020240528094033.png)

# Probability Density Function
Three parameters are required :
- incoming light direction
- a point on the surface
- outgoing direction as output storage
```cpp
// for example, the function signature can look like
void pdf(vec3 in_direction, vec3 point_on_surface, /*storage*/ vec3 out_direction)
```
 
This tells us about what the probability of outgoing direction in given interval.
Formally, it could be written :
$$f_r(\vec{w},x,\vec{w}^{\prime})$$
where omega with prime is outgoing direction.

This is our BRDF model.

What if a light transmit the surface? BTDF(T of transmittance) can be defined for this in this case.
![](../../../../images/Pasted%20image%2020240528095035.png)


Roughly, BSDF or BxDF(Bidirectional Scattering Distribution Function) is combination of BRDF and BTDF
![](../../../../images/Pasted%20image%2020240528100208.png)

## Optics
Every material is composed of numerous atoms.
But Atom is composed of 99% empty space (football at center of football field)
Then why everything isn't transparent if they are composed of empty space ?

The answer is 'absorption'.
There are electrons around nucleus in atom and they can absorb photons (a photon is a light in the perspective of particle. a light has both of particle-like property and wave-like property at the same time.)

Then why glass is transparent ?
It is because the orbit of electrons are different dependent on the materials.
In the case of glass-like materials, the orbit of electrons is too far to get enough energy to jump into different orbit. (note that this is described in the context of visible light spectrum)

# BRDF properties

1. Helrnholtz-reciprocity
	The direction of the ray of light can be reversed. That means we can swap incoming and outgoing direction.
	$$\vec{w}, \vec{w}^\prime : f_r(\vec{w},x,\vec{w}^\prime)=f_r(\vec{w}^\prime,x,\vec{w})$$

2. Positivity - one of axioms of probability. (because BRDF is pdf)
3. Energy conservation
	The most important property.
	It is impossible to reflect more than incoming light energy.
	$$\int_{\Omega}f_r(\vec{w},x,\vec{w}^\prime)\cdot cos\theta\space d\vec{w}^\prime \le 1$$
	Based on above equation, we add up the incoming energy from all directions (while taking light atteunation into account)
	If the result of integral is 1, all light is reflected.
	Otherwise(less than 1) , some of light absorbed
	More than 1 is impossible (axiom of probability)

# The rendering equation

![](../../../../images/Pasted%20image%2020240528100953.png)

Things to know :
- object can absorb light energy
- object can reflect light
- object can emit light

Hence, Intuitvely 'light exit the surface = Emitted light + reflected light'

We can write this formally in the beautiful rendering equation.
$$L_o(x, \vec{w})=L_e(x, \vec{w})+\int_{\Omega}L_i(x, \vec{w}^\prime)\cdot f_r(\vec{w},x,\vec{w}^\prime)\cdot cos\theta\space d\vec{w}^\prime$$
- $L_o$ : exiting radiance in point $x$ towards direction $w$
- $L_e$ : emitted radince in point x towards direction $w$
- $\int_{\Omega}{...\space}dw\prime$ : sum of incoming radiance from all $w\prime$
- $L_i$ : incoming radiance from direction $w\prime$
- $f_r$ : BRDF
- $cos\theta$ : light atteunation

Note that we only integrate the equation against hemi-sphere, not the full sphere.
It doesn't make sense the integrate a light outgoing in the invisible direction(over 90degree) to viewer. (we take care of this using cosine term.)

But this equation is almost impossible to solve in closed form in most of complex scenes. (At the same time, the brightness of point $x$ depends on a lot of other points in the scene. Therefore, the algorithm is recursive and takes a huge amount of memory)
	Even the integral is also infite-dimensional

An example of light path.
![](../../../../images/Pasted%20image%2020240528102445.png)

# Reference
[Tu Wien rendering course](https://www.youtube.com/watch?v=4gXPVoippTs&list=PLujxSBD-JXgnGmsn7gEyN28P1DnRZG7qi&index=3)