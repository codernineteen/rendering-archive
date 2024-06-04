
When a ray hit one of points on surface, A ray may be reflected on the surface or refracted toward the downside of surface depending on which material properties is applied to the surface.

- blue marked line - reflection
- yellow marked line - refraction
![](../../../../images/Pasted%20image%2020240603151123.png)

Then which of effect is stronger ?
In the above image, amount of refraction is stronger than reflection.

Then we can write the vector notations on the image.
- $\vec{N}$ : surface normal
- $\vec{L}$ : incoming light direction
- $\vec{R}$ : outgoing light direction
- ${\theta_i}$ : an angle between surface normal and incoming light direction
- ${\theta_r}$ : an angle between surface normal and outgoing light direction
- $-\vec{N}$ : reversal of surface normal
- $\theta_t$ : an angle between reversed surface normal and refracted ray
![](../../../../images/Pasted%20image%2020240603151447.png)

There are simplifed version of equation that represent above relationship between surface and incoming light.

- Schlick's approximation : 
	This equation gives us a probability of reflection ($R(\theta)$)
$$R(\theta) = R_0 + (1-R_0)(1-cos\theta)^5$$
	- $R(\theta)$ : probability of reflection when the incident ray angle is $\theta$
	- $R_0$ : probabillity of reflection on normal incidence (in this case, $\theta=0$ )
		$$R_0 = (\frac{n_1 - n_2}{n_1 + n_2})^2$$
	, where $n_1$ is the index of refraction of the new medium

If it is interactions with Air/vacumm medium, then
		$$R_0 = (\frac{n_1 - 1}{n_1 + 1})^2$$ 

It is obvious that the probability of refraction (transmission) will be $T(\theta)$, which is equal to $1- R(\theta)$


By replace $\theta$ with 0 degree and 90 degree, we can get an intuition for two cases.
1. When $\theta$ is close to 0 , refraction probability becomes very high
2. On the other hand, If $theta$ is close to 90 degree, reflection probability goes up.

- An overview of how $R(theta)$ changes along the incoming angle. (in the case of air-glass interaction)
![](../../../../images/Pasted%20image%2020240603153820.png)

For the interaction with vacumm, vacumm has an index of $n_vacumm$ which is equal to 1.
By definition, there is nothing in vacumm space.
So we may expect that there will be no reflection at all.

But if we compute the $R_0$ with the index of $n_{vacumm}$ and replace the term with one in the Schlick's approximation, we can plot some kind of graph like below.
![](../../../../images/Pasted%20image%2020240603154443.png)

Obviously, there is a chance for reflection in the graph.

As the equation name has, It is 'approximation', not the true solution.
So Schlick's approximation is used to efficiently calculate vacumm-medium type of interactions, but It doens't mean that it is physically true.

The original Fresnel equation is :
$$R_S(\theta) = |\frac{n_1cos\theta - n_2\sqrt{1-(\frac{n1\cdot sin(\theta)}{n2})^2}}{n_1cos\theta - n_2\sqrt{1-(\frac{n1\cdot sin(\theta)}{n2})^2}}|^2$$
If we replace $n_1, n_2$ as 1, we can derive $R_s(\theta)$ is exactly equal to 0 (Therefore ray is just refracted)
