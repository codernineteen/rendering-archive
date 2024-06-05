
An abstraction for ray :
![](../../../../images/Pasted%20image%2020240605131959.png)
- The magnitude of direction vector should be 1 (That is, it should be normalized)

In the equation form, a ray can be represented as :
$$\vec{r}(t) =  \vec{o} + t\cdot\vec{d}$$ where $t$ represent the distance of the ray.

When we represent an geometry with a function, we can show it as implicit equation something like below.
$$f(x,y) = 0$$
For example, a 2D circle which has a center (a,b) coordinates and radius 'r' will be shown as :
$$(x-a)^2 + (y-b)^2 - r^2 = 0$$

In the meanwhile, parameteric equation can be replaced with each of variables like :
- $x = x(t)$
- $y = y(t)$
- $\vec{r} = r(t)$


# Intersection scenario in ray-sphere
1. Ray is intersected with two points on the sphere (2 solution)
	![](../../../../images/Pasted%20image%2020240605132647.png)

2. When the hit point is tangented on the boundary of sphere (1 solution)
	![](../../../../images/Pasted%20image%2020240605132703.png)

3. When a ray missed
	![](../../../../images/Pasted%20image%2020240605132826.png)

# How to find roots against intersection equation

- p : point on sphere surface
- c : center of sphere

Then,
$$(\vec{p} - \vec{c})\cdot(\vec{p} - \vec{c})=r^2$$
and a ray is :
$$\vec{r}(t)=\vec{o}+t\cdot\vec{d}$$
By substituting $\vec{r}(t)$ into the place of $\vec{p}$

$$(\vec{r}(t) - \vec{c})\cdot(\vec{r}(t) - \vec{c})=r^2$$
$$(\vec{o}+t\cdot\vec{d} - \vec{c})\cdot(\vec{o}+t\cdot\vec{d} - \vec{c})=r^2$$Because we're interested on the value of 't', let's reorder the variables
$$(\vec{o}- \vec{c} +t\cdot\vec{d})\cdot(\vec{o}- \vec{c}+t\cdot\vec{d})=r^2$$
Then Substitute $\vec{o}- \vec{c}$ into a new temporary variable $\vec{A}$

$$(\vec{A} +t\cdot\vec{d})\cdot(\vec{A}+t\cdot\vec{d})=r^2$$

Now we can proceed the equation algebraically,
$$\vec{A}\cdot\vec{A} + 2t(\vec{A}\cdot\vec{d}) + t^2(\vec{d}\cdot\vec{d})-r^2 = 0$$
Note that both $\vec{A}$ and $\vec{d}$ are three dimensional vector, so dot product of two vector can satisfy communicative property. ($\vec{A}\cdot\vec{d} =\vec{d}\cdot\vec{A}$)

Finally, let's clear the equation in terms of 't'
$$(\vec{d}\cdot\vec{d})t^2 + 2(\vec{A}\cdot\vec{d})t + (\vec{A}\cdot\vec{A}-r^2) = 0$$
To solve this equation, we can use quadratic formula :
$$x =\frac{-b \pm \sqrt{b^2-4ac}}{2a}$$, when the equation is $ax^2 +bx+c=0$

Note that the number of roots is dependent on determinant expression $D = b^2 - 4ac$
- if $D < 0$ , the square root term is under zero and it is imaginary -> no root
- if $D = 0$, there is exactly one root.
- if $D > 0$ , there will be two roots against the given equation

Especially, when there are more than one solution, we can ignore the bigger root because it is occluded by the front face of the mesh.
