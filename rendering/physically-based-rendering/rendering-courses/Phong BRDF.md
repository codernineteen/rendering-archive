
# Illumination model

Note : $\vec{u}\cdot\vec{v} = |u||v|cos\theta$ 

- ambient 
	This is not related to physics at all, but it makes a good default appearance.
	we do product ambient coffiecient($K_a$) to Intensity $I_a$.
	Roughly, this is a default color of certain object.
	$$I = K_aI_a$$
	

- diffuse
	The diffuse and other coefficients may take different values for different wavelengths (for example, an abject absorb most of greem light,  but reflect incoming red light in the meantime)
	$$I =K_{d,\lambda}(\vec{L}\cdot\vec{N})$$
	Diffuse term is not related to viewer's direction. So Even if we move our head, the diffuse term is constant dependent on incoming light direction and the object's normal vector.

- specular
	prodcuct result of a vector pointing to viewer and reflected direction of the light.
	$n$ is shininess.
	$$I=K_S(\vec{V}\cdot\vec{R})^n$$ 
	Unlike diffuse, Specular reflection is dependent on viewer's direction as it denoted in equation.
	![](../../../../images/Pasted%20image%2020240530120310.png)

- Phong BRDF : Ambient + diffuse + specular
	Simpler rendering equation in Phong BRDF
	$$I = K_aI_a + I_i(K_{d,\lambda}(\vec{L}\cdot\vec{N}) + K_S(\vec{V}\cdot\vec{R})^n)$$
	
	Phong BRDF is physical plausible illumination model.
	![](../../../../images/Pasted%20image%2020240530115810.png)
	
	Note that the phong BRDF only accounts for direct illumination. (For reference, If we take care of indirect light on the surface, we call it as 'global illumination')
	
	Also if there are multiple light sources, we should modify the equation.






Reference
[Tu Wien rendering course](https://www.youtube.com/watch?v=4gXPVoippTs&list=PLujxSBD-JXgnGmsn7gEyN28P1DnRZG7qi&index=3)