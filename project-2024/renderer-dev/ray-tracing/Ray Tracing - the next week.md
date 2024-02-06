
# Motion Blur
In a real camera, the shutter open for a short time interval. (world and objects may move during the time).
-> we need to **abstract this camera shot** into our class.

## Space time ray tracing
Get a random estimate of a single photon by sending a single ray at some random instant in time while the shutter is open.


## Managing time

Aspects of time :
1. one shutter opening -> next shutter opening
2. how long the shutter stays open for each frame
	Each frame can have its own shutter speed.

To render a sequence of images, we need to setup frame-to-frame period, shutter/render duration and total number of frames.

Also If the world is dynamic, we would need to add a method to `hittable` so that every object could be made aware of the current frame's time period.

In much simpler model that has a single frame, we only care a time slice from 0 to 1.
Now we need to modify the camera class to launch rays with random times in `[0,1]`


## Update the camera to simulate motion blur
Generate a random time in interval from 0 to 1 when we create a ray.


## Bounding Volume Hierachies
Ray-object intersection is the main time-bottleneck in a ray tracer. 

## key idea
---
We have a volume that fully encloses(bounds) all the objects.
-> Any ray that misses the bounding volume will never hit all objects enclosed by the volume.

Note that any object exists in one bounding volume, but bounding volumes can be overlapped each other.

- red bounding volume - contains red objects
- blue bounding volume - contains blue objects
- purple bounding volume - contains red and blue volumes.
![bounding volume hierachy](https://raytracing.github.io/images/fig-2.01-bvol-hierarchy.jpg)


## Axis-Aligned Bounding Boxes(AABBs)

We need 
- a way to make good divisions
- a way to intersect a ray with a bounding volume

Axis-aligned Bounding rectangular parallelpiped, shortly AABBs, works better than the alternatives in general cases.

All we need to know is whether we hit the volume or not until we reaches the leaf node (actual object) in the hierachies.

A ray can be represented as :
$$P(t) = A+tb$$, where A is origin of ray, b is its direction.

Because we're not interested in hit point on plane, we can only consider whether the ray lives in between two planes.
For example, planes on x-axis will be :
- $x = x_0$, $x = x_1$
Then we can solve this equation with a ray equation like this way :
$$x_0 = A_x + t_0b_x$$, where $A_x,b_x$ are x-component of the ray.
![ray-intersection-with-plane](https://raytracing.github.io/images/fig-2.03-ray-slab.jpg)

If we expand this 1-dimensional intersection into 2-dimensional,
![2d-intersection](https://raytracing.github.io/images/fig-2.04-ray-slab-interval.jpg)
We can see that overlapping of each t-intervals only happen when there is a hit.

- Algorithm 
	- compute x-axis aligned interval
	- compute y-axis aligned interval 
	- compute z-axis algined interval
	- check if the intervals overlapped with each other

But we have two problems to solve.
1. When the ray travelling in the negative direction on certain axis , the interval will be inverted.
2. If denomitor 'b' is zero, it causes division by zero problem.
	- When the b is only zero, the interval components t0 and t1 will be both -inf or inf in c++ -> we can use min, max approach here.
		ex) Below code prints true in c++
		```cpp
    float denominator = 0.0;
    float numerator = 10.0;
    float quotient = numerator / denominator;
    
    cout << (quotient == numeric_limits<float>::infinity()) <<endl;
		```
		
## optimized AABB hit method
This is go-to hit method by the reference book
```cpp
class aabb {
  public:
    ...
    bool hit(const ray& r, interval ray_t) const {
        for (int a = 0; a < 3; a++) {
            auto invD = 1 / r.direction()[a];
            auto orig = r.origin()[a];

            auto t0 = (axis(a).min - orig) * invD;
            auto t1 = (axis(a).max - orig) * invD;

// if inverted direction is lower than zero that is negaitve, it means that the direction of ray is negative based on the world axis
            if (invD < 0)
	            // then swap the elements of the interval
                std::swap(t0, t1);

            if (t0 > ray_t.min) ray_t.min = t0;
            if (t1 < ray_t.max) ray_t.max = t1;

            if (ray_t.max <= ray_t.min)
                return false;
        }
        return true;
    }
    ...
};
```


# BVH volumes
The complicated part is to build a heirachy structure of volumes.
We follow simply :
1. randomly choose an axis
2. sort the primitives 
3. put half in each subtree
	If a list comming in is two elements, put one in each subtree and end the recursion.

- hit detection
```cpp
    bool hit(const ray& r, interval ray_t, hit_record& rec) const override {
    // if the ray doesn't hit super bounding volume, return false
        if (!box.hit(r, ray_t))
            return false;
	// if a ray hit the volume, we check if there are hits on sub-volumes
        bool hit_left = left->hit(r, ray_t, rec);
        bool hit_right = right->hit(r, interval(ray_t.min, hit_left ? rec.t : ray_t.max), rec);

        return hit_left || hit_right;
    }
```


# Texture mapping
Overview of texture mapping : 
a vertex <-> texture coordinate <-> texture image

## solid texture (spatial texture)
a solid texture depends only on the position of each point in 3D space.
It is coloring space itself, rather than coloring object in that space.

# Sphere texture mapping

To map texture with a spherer, we can apply the concepts of longitude and langitude.
About u, v coordinates on texture, we can think of these formula
Horizontally,
$$u=\frac{\phi}{2\pi}$$
Vertically,
$$v=\frac{\theta}{\pi}$$

And about a certain point on the sphere,
$$y=-cos(\theta)$$
$$x=-cos(\phi)sin(\theta)$$
 $$z=sin(\phi)sin(\theta)$$
- An intuitive image description for this formulas (radius of sphere is equal to 1)
![](../../../images/Pasted%20image%2020240206103640.png)

Then, To extract $\theta$, $\phi$ from the point (x,y,z) , we need to invert the formulas

For $\phi$,
$$\phi = atan2(z, -x) = atan2(-z, x) + \pi$$
We use last formula to avoid that 'u' becomes discontinuous.

For $\theta$,
$$\theta = arccos(-y)$$

Now we get a (u,v) coordinates for points on the sphere.
The next step we can take is to extract a pixel value from an image which is corresponded to the coordinates.

Not to harm the resolution of image, we can set u, v as fractional position.
$$u = \frac{i}{N_x-1}$$
$$v=\frac{j}{N_y-1}$$
, where an image has $N_x$ by $N_y$ size.
So obviously the pixel (i, j) will be :
$$u * (N_x-1) = i$$
$$v * (N_y-1) = j$$
# Reference 
https://raytracing.github.io/books/RayTracingTheNextWeek.html#motionblur/puttingeverythingtogether