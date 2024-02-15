
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

## 2.2 Stratified samples
If we make the estimation steps to run forever, we can observe the estimation becomes near $\pi$ very quickly, but then more slowly become close to real $pi$. (*Law of Diminishing Returns*).

Instead picking up a random point from a single whole square, we can make several sample area target(stratification) and pick the random number from the areas.
![](../../../../images/Pasted%20image%2020240214125735.png)

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
![](../../../../images/Pasted%20image%2020240214130945.png)

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


# 3. One dimensional Monte Carlo Integration


We could estimate a $\pi$ by solving for the ratio of the area of the circle and the aread of the inscribing square.
$$\frac{area(circle)}{area(square)}=\frac{\pi}{4}$$
The more we added more random points , the more correct the estimation becomes.

Although we don't know the area of the unit circle, we still can solve for it using the ratio.
Assuming that we know the radius,
$$\frac{area(circle)}{area(square)}=\frac{\pi}{4}$$
$$\frac{area(circle)}{(2r)^2}=\frac{\pi}{4}$$
$$area(circle)=\frac{\pi}{4}\times 4r^2$$
$$area(circle) = \pi r^2$$
If we choose r as 1, Estimating pi has same effect to estimate area of circle.

## 3.1 Expected value

Monte carlo algorithm in general perspective :

Assuming that
- A list of values $X$ that contains members $x_i$ : $X=(x_0, x_1, ..., x_{N-1})$
- A continuous function $f(x)$ that takes members from the list
	$y_i=f(x_i)$
- A function $F(X)$ that takes the list $X$ as input and produces the list $Y$ as output 
	$Y=F(X)$
- Y has memebers $y_i$
	$Y=(y_0, y_1, ..., y_{N-1})=(f(x_0),f(x_1),...,f(x_{N-1}))$

Then the average of $Y$ is :
$$average(Y)=E[Y]=\frac{1}{N}\sum_{i=0}^{N-1}y_i=\frac{1}{N}\sum_{i=0}^{N-1}f(x_i)=E[F(X)]$$

If the values of $x_i$ are chosen randomly from a continuous interval $[a,b]$ such that $a \le x_i \le b$ for all values of i, then $E[f(X)]$ will approximate the average of the continuous function $f(x')$ over the inverval $a \le x' \le b$

$$E[f(x')| a\le x' \le b]\approx E[F(X)|X=\{x_i | a \le x_i \le b\}] \approx E[Y=\{f(x_i) | a\le x_i \le b\}]\approx \frac{1}{N}\sum_{i=0}^{N-1}f(x_i)$$
and we can accomplish this equation when we take the number of samples $N$ and the $N$ goes to infinity,
$$ E[f(x') | a\le x' \le b]=\lim_{N\to \inf} \frac{1}{N}\sum_{i=0}^{N-1} f(x_i)$$
The expected value of continuous function $f(x_i)$ can be perfectly represented by summing an infinite number of random points within the interval.


Not only we can sample random points to solve for the expected value over an interval, but also **we can choose where we place our sampling points**.

From left boundary of given interval $[a,b]$ , i-th point can be denoted as :
$$x_i = a+i\Delta x$$
Also we can choose equally spaced $\Delta x$
$$\Delta x=\frac{b-a}{N}$$

Depending on above equations, solving for their expected value :
$$ E[f(x') | a\le x' \le b] \approx \frac{1}{N}\sum_{i=0}^{N-1} f(x_i)$$
$$ E[f(x') | a\le x' \le b] \approx \frac{\Delta x}{b-a}\sum_{i=0}^{N-1} f(x_i)$$
$$ E[f(x') | a\le x' \le b] \approx \frac{1}{b-a}\sum_{i=0}^{N-1} f(x_i)\Delta x$$
By taking the limit as N approches positive infinity, the expected value is equal to regular integral of $f(x)$.
$$ E[f(x') | a\le x' \le b] = \lim_{N \to \inf}\frac{1}{b-a}\sum_{i=0}^{N-1} f(x_i)\Delta x =\frac{1}{b-a}\int_{a}^{b}f(x)dx$$
In calculus, Integral has meaning an 'Area' under a curve over given interval.
$$area(f(x),a,b) = \int_{a}^{b}f(x)dx$$

Finally we can make a relation between 'expected value' and 'area'
$$E[f(x) | a\le x \le b] = \frac{1}{b-a}\cdot area(f(x),a,b)$$

Now we got to know that 'counting the number of points that fall inside of an object' isn't the only way to measure its average or area.

## 3.2 Integrating example. closed form VS monte carlo estimation

- closed form
$$I = \int_{0}^{2}x^2dx = \frac{1}{3}x^3|^2_0 = \frac{1}{3}(2^3-0^3) = \frac{8}{3}$$

- Monte carlo estimation
	$$I = area(x^2, 0, 2) = (2-0)\cdot E[x^2|0 \le x \le 2]$$
	Then we can follow this code.
```cpp
int a = 0, b = 2; // interval
int N = 1000000;  // sample points
auto sum = 0.0;
for(int i = 0; i < N; i++)
{
	auto x = random_double(a,b);
	sum += x*x; // x^2
}
int I = (b-a)*(sum/N);
```

Because there exist some functions that is more easier to solve its integral with Monte carlo or functions that can't be solved in closed form, Monte carlo algorithm is really useful tool to solve those kind of functions.

## 3.3 Density functions

`ray_color` function in previous series has a major problem, which is that small light sources create too much noise.

Light sources only sampled if a ray scatters toward them -> this can be unlikely for a small light.

When background color is black, the only **real sources of light** in the scene are from the light objects placed in the space. (In our case, it was quad geometry with emissive material.)

Assuming that there are two rays that intersect at nearby points on a surface,
One ray may be randomly reflected toward light object and the other may not.
In this case, The ray reflected toward light object will have very bright color.
On the other hand, the other ray will appear a very dark color.

This is strange because they intersect at nearly same point , but have huge difference in their color representation.

In real world, a light starts from light source and it reaches our eye. 
The ray may start with a very bright intensity and would lose energy with each successive bounce around the scene.
Now think about the case when a ray is forced to bounce toward camera as soon as it could.
The ray would appear inaccurately bright because it hadn't been dimmed by successive bounces as it should have done.

How can we remove this inaccuracy by downweighting those samples to adjust for the oversampling?
For this, We need to understand what the 'probability density funciton' is first.

##
# Reference
 - This markdown was writtien entirely based on :
	[_Ray Tracing: The Rest of Your Life](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html)
