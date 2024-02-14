
# 1. Motivation
From 'the weekend' and 'the next week' series, we learned most of things to implement real ray tracer in a scale of hobby project.
However, we need deeper knowledge if we want to get a very serious ray tracer.


# 2. A Simple Monte Carlo program
Two kinds of randomized algorithmms :
- Monte Carlo - **may produce** the correct result.
	- based on statistical estimate of an answer.
		-> The longer we run it, The more accurate the result becomes.
- Las Vegas - always produces the correct result.
	- quick sort is one of the examples using LV algorithm.

## 2.1 estimating $\pi$

Let's think about a circle is originated at (0,0) and it has one as its radius.
Then there is a square who wrap the circle while the four sides of the square is tangent to circle (ranged between -1 and 1).

If we pick a random point inside a square, the probability that the point will be placed in the circle is :
$$\frac{\pi r^2}{(2r)^2} = \frac{\pi}{4}$$
Now can we guess the Pi by code?
Because we can cancel out radius term from equation, we can assume the radius as 1.
```cpp
int N = 1000000;
int inside_circle = 0;

double x, y;
for (int i = 0; i < N; i++) {
	auto x = random_double(-1, 1); // randomly generated float number by certain seed
	auto y = random_double(-1, 1);
	if (x * x + y * y <= 1) inside_circle++; // if the square of length is less or equal to one -> inside circle
	
	std::cout << std::fixed << std::setprecision(12);
	std::cout << "Estimate of pi: " << 4 * inside_circle / (double)N << std::endl;
}
```
In most cases, the output of estimation shows a number nearly same to true Pi.

## 2.3 Stratified samples
If we make the estimation steps to run forever, we can observe the estimation becomes near $\pi$ very quickly, but then more slowly become close to real $pi$. (*Law of Diminishing Returns*).

Instead picking up a random point from a single whole square, we can make several sample area target(stratification) and pick the random number from the areas.
![](../../../images/Pasted%20image%2020240214125735.png)

Let's convert this into the code.
```cpp
int inside_circle = 0;
int inside_circle_stratified = 0;

int sqrt_n = 1000;
for (int i = 0; i < sqrt_n; i++) {
	for (int j = 0; j < sqrt_n; j++) {
		// regular estimate
		auto x = random_double(-1, 1);
		auto y = random_double(-1, 1);
		if (x * x + y * y < 1) {
			inside_circle++;
		}

		// stratified estimate
		auto x_stratified = 2 * ((random_double() + i) / sqrt_n) - 1; // random_double() returns a value in [0, 1)
		auto y_stratified = 2 * ((random_double() + j) / sqrt_n) - 1;
		if (x_stratified * x_stratified + y_stratified * y_stratified < 1) {
			inside_circle_stratified++;
		}
	}
}
```
- result 
	we can see that the not only stratified method has better result, but also it converges with a better asymptotic rate.
![](../../../images/Pasted%20image%2020240214130945.png)

But this effect reduces while the dimension of the problem goes up and Ray tracing is a very high-dimensional algorithm. (each reflection adds two new dimensions $\phi_o$ and $\theta_o$)

Instead of stratifying output angles of reflected ray, let's try to stratify the locations of the sampling positions around each-pixel location.

- our previous sampling method
```cpp
...
				for (int sample = 0; sample < samples_per_pixel; sample++) {
					Ray r = get_ray(i, j);
					pixel_color += ray_color(r, max_depth, world);
				}
...

Ray get_ray(int i, int j) const {
	auto pixel_center = pixel00_loc + (i * pixel_delta_u) + (j * pixel_delta_v);
	auto pixel_sample = pixel_center + pixel_sample_square();

	auto ray_origin = (defocus_angle <= 0) ? center : defocus_disk_sample();
	auto ray_direction = pixel_sample - center;
	auto ray_time = random_double();

	return Ray(center, ray_direction, ray_time);
}

Point3 defocus_disk_sample() const {
	auto p = random_in_unit_disk();
	return center + (p[0] * defocus_disk_u) + (p[1] * defocus_disk_v);
}

Vec3 pixel_sample_square() const {
	auto px = -0.5 + random_double();
	auto py = -0.5 + random_double();
	return px * pixel_delta_u + py * pixel_delta_v;
}

```


# Reference
 - This markdown was writtien entirely based on :
	[_Ray Tracing: The Rest of Your Life](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html)
