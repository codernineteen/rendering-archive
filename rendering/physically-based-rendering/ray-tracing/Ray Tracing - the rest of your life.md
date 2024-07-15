
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

A density function is continuous version of a histogram.

- histogram example
![](../../../../images/Pasted%20image%2020240216102018.png)

Based on above histogram, we can think of three possible cases.
- more items , but same number of bins -> each bin has higher frequency
- more items and more bins -> each bin has lower frequency
- increase bins to infinity -> now we have an infinitie number of zero-frequency bins
So If number of bins diverges to positive infinity, the probability of certain bin typically is zero. -> we need another representaiton.

To solve for this, we can replace the histogram with a discrete density function.
The difference between discrete function and discrete density function is that the latter one normalizes y-axis to a percentage of the total, which is called density.

- Conversion from discrete function to discrete density function.
$$Density \space of \space Bin \space i = \frac{n(bin_i)}{n(total)}$$
, where $n(A)$  is the number of type A.
Now we got a 'discrete' density function and we can convert this into 'continuous' density function.

But how?
A Density of Bin i is corresponded to a single bin.
Then, we can take any two ranges between H and H' where H <= H'.
If we divide up fraction of trees between height H and H' by the range, we can get a bin density against certain range.
The choice of ranges are infinite, so we can represent this as continuous form.
$$Bin\space Density=\frac{Fraction \space of \space trees \space between \space H \space and \space H'}{H-H'}$$

Back again, the ratio of number of items in Bin i over total items can be considered as probability of bin i
$$p(Bin_i)=\frac{n(Bin_i)}{n(total)}$$
By combining density function and probability function altogether, we can get any likelihood that any given tree has a height that places within any arbitrary span of multiple bins :
$$p=Bin \space Density\cdot(H-H')$$

In short, a PDF is a continuous function that can be integrated over to determine how likely a result is over an integral.
With given PDF $f_X(x)$
$$\int_{-\inf}^{inf}f_X(x)dx=1$$
$$P(a\le x \le b)=\int_{a}^{b}f_X(x)dx$$
and the probability of a certain variable will always be zero.
$$P(X=a)=P(a\le x \le a)=\int_{a}^{a}f_X(x)dx=0$$

Again, Remeber that PDF only gives us a probability within an interval, not about single variable




## 3.4 A Linear PDF example
![](../../../../images/Pasted%20image%2020240219114404.png)
- Remind that PDF is a probability function
- area(p(r), 0, 2) = 1 (integral between 0 and 2)
- $p(r) = C\cdot r$ , where  $p(r)$ is linear function and $C$ is arbitrary constant
- Derivation for C :
	$$1=area(p(r), 0, 2)$$
	$$=\int_{0}^{2}C\cdot rdr$$
	$$=C\int_{0}^{2}rdr$$
	$$=C(\frac{2^2}{2}-\frac{0}{2})$$
	Therefore,
	$$C=\frac{1}{2}$$
- $P(r|x_0 \le r \le x_1)$ is equal to
	- $area(p(r), x_0, x_1)$
	- $\int_{x_0}^{x_1}\frac{r}{2}dr$
- $P(r=x) = 0 = \int_{x}^{x}p(r)dr$




## 3.5 choosing samples

PDF needs to find a best way to sample a scene so that we wouldn't get very bright pixels next to very dark pixels.

- uniformly random sampling is inaccurate
- Steering samples artifically also causes inaccurate result.

Then How do we generate random samples with PDF?
If our PDF is uniform over a domain `[0, 10]` , then we can generate a perfect sample like :
`random_double() * 10.0`, where random_double generate a random number from uniform distribution ranged between 0.0 and 1.0.

**But in most cases, our interest is non-uniform**.
Assuming belows,
- a distribution is defined by PDF
- there is a uniform random number generator
We need to find a way to convert the generator into non-uniform random number generator.


Now let's assume that **there exists a function $f(d)$ that takes uniform random input and produces a non-uniform distribution weighted by PDF.**
Back again to our linear PDF example, we know that the probability of certain interval will be high at near 2 than near zero.

However what is the value that splits the probability in half? 
In other words, what is the value that a random number has a 50% chance for intervals `[a, x]` and `[x, b]` where total range is `[a, b]`.

We can solve this by following :
$$50\%=\int_{0}^{x}\frac{r}{2}dr=\frac{r^2}{4}|^x_0=\frac{x^2}{4}$$
$$\frac{x^2}{4}=0.5$$
$$x=\sqrt{2}$$

As an approximation, we can create a $f(d)$ which generates a number in `[0, sqrt(2)]` when d is less than 0.5 and a number in `[sqrt(2), 2]` when d is greater than 0.5.

- Code 
```cpp
double f(double d)
{
	if(d <= 0.5)
		return sqrt(2.0) * random_double();
	else 
		return sqrt(2.0) + (2.0-sqrt(2.0))*random_double();
}
```
So we got a new distribution.
![](../../../../images/Pasted%20image%2020240219121750.png)

It is really easy to solve the intergration above in analytical way.
But As being mentioned before, there will be functions that we don't and can't solve for the integrations.
- an example that we don't want to solve analytically.
	$$p(x)=e^{\frac{-x}{2\pi}}sin^2(x)$$
![](../../../../images/Pasted%20image%2020240219122007.png)

In this case, we need to solve it experimentally by making the result close to real value.

A code to solve this in experimental way.
```cpp

struct sample {
	double x;
	double p_x;
};

// comparator for sorting by x in ascending order
bool compared_by_x(const sample& a, const sample& b) {
	return a.x < b.x;
}

int main() {
	int N = 10000;
	double sum = 0.0;

	std::vector<sample> samples;
	for (int i = 0; i < N; i++) {
		auto x = random_double(0, 2 * pi);
		auto sin_x = sin(x);
		auto p_x = exp(-x / (2 * pi)) * sin_x * sin_x; 
		sum += p_x;

		sample current_sample = { x, p_x };
		samples.push_back(current_sample);
	}

	std::sort(samples.begin(), samples.end(), compared_by_x);
	double half_sum = sum / 2.0;
	double halfway_point = 0.0;
	double accum = 0.0;
	for (int i = 0; i < N; i++) {
		accum += samples[i].p_x;
		if (accum >= half_sum) {
			halfway_point = samples[i].x;
			break;
		}
	}
	std::cout << std::fixed << std::setprecision(12);
	std::cout << "Average = " << sum / N << '\n';
	std::cout << "Area under curve = " << 2 * pi * sum / N << '\n';
	std::cout << "Halfway = " << halfway_point << '\n';
}
```

I've shown only one partitioning. But we can do this recursively based on our choose for depth.
1.  solve for halfway point of a PDF
	1-1 Recurse into lower half, repeat step 1
	1-2 Recurse into upper half, repeat step 1

The main time complexity is depending on sorting algorithm whose best complexity is $O(n \log n)$ (quick sort.)

There is a research for this topic : *Metropolis-Hastings algorithm*

## 3.6 Approximating distributions

- Cumulative Distribution Function
ex) $f(x) = e^{\frac{-x}{2\pi}}sin^2(x)$
From 0 to $2\pi$ , the area under curve is equal to one(100%).
From 0 to about 2.016(halfway point), the area under curve is 0.5(50%).

Because we only take care about distribution between 0 and $2\pi$, any interval 'x' $\notin [0, 2\pi]$ is equal to zero. 

By generalizing the CDF, we can represent it as a function $P(x)$.
$$P(x) = \int_{-\infty}^{x}p(x')dx'$$
or,
$$P(x) =area(p(x'), -\infty, x)$$

For summary,
- PDF : a probability function that explains how likely an interval of numbers is to be chosen.
- CDF : a distribution function that explains how likely all numbers smaller than its input is to be chosen.

- PDF -> CDF : integrate PDF from $-\infty$ to x
- CDF -> PDF : take derivative of CDF.

Let's take an previous linear PDF example.
If we take integral of the PDF, we can get CDF about some intervals.
$$P(r) = \begin{cases}
0 \space\space\space\space\space\space r<0\\
\frac{r^2}{4} \space\space\space\space 0 \le r \le 2\\
1 \space\space\space\space\space\space 2 < r\\
\end{cases}$$
With this example, $P(1) = 0.25$ means that any random variable from our PDF has a 25% chance of being 1 or lower.

We want a function $f(d)$ to take a uniform distribution(0~1) and return a random value depending on a CDF '$P(r)$'.

By aligning some cases, we can get a intuition for $f(d)$.
- $f(0.25) = 1$ , any random value is less or equal to one with a 25% chance.
- $f(0.5) = \sqrt{2}$ , any random value is less or equalt to $\sqrt{2}$ with a 50% chance.
- Hence, $f(P(x)) = x$

Although a CDF P(x) is accumulated til certain point, it still follows characteristics of probability,  "sum of probabilities is equal to 1".
Then any interval defined in CDF will return a value between 0 and 1, we know that this value will be input of f(d).

Finally f(d) takes an input 0~1 and it returns a certain value between 0~2 in the given case.

We can represent f(d) in a more concise way by inversing P(x).
$$f(P(x)) = x$$
$$f(d)=P^{-1}(x)$$


## 3.7 Importance sampling

Example)
$$I = \int_{0}^{2}x^2dx$$
When we solved this using Monte carlo approach, we were implicitly using uniform PDF between 0 and 2. 
- PDF = $\frac{1}{2}$ over te range $[0,2]$
- $P(x) = \frac{x}{2}$ 
- $f(d) = P^{-1}(x) = 2d(\because P(x) = d, \frac{x}{2}=d, x = 2d)$ 
With above equations, we can use explicit PDF now.

If we write down the approximation of Integration in code,
```cpp
double f(double d) {
    return 2.0 * d;
}

double pdf(double x) {
    return 0.5;
}
int main() {
    int N = 1000000;
    auto sum = 0.0;
    for (int i = 0; i < N; i++) {   
/*
a sample of the function x^2 within the distribution [0,2]
'f' is used to randomly select samples from this distribution
*/    
	    auto x = f(random_double());
        sum += x*x / pdf(x);    
    }
}
```

In Monte carlo estimation example, we followed this equation :
$$I = area(x^2, 0, 2) = (2-0)E(X)$$
So in code,
```cpp
I = (b-a) * (sum / N);
```
where (sum / N) is expected value and (b-a) is target interval.

However, we don't need to multiply the average by 2 anymore.
We need to account for the nonuniformity of PDF of $x$.
- The PDF will steer samples toward specific parts of the distribution, which will cause us to converge faster,but at the cost of introducing bias.
- A bias in the scene causes inaccurately bright scene.
- To reduce bias, 
	- down-weight where we sample more frequently
	- up-weight where we sample less frequently
	Following above weighting rule, the weighting function should be proportional to $\frac{1}{pdf}$

Now let's try to solve for the integral using 'a linear PDF($p(r) = \frac{r}{2}$)' which is not uniform.
```cpp
double f(double d) {
	return sqrt(4.0 * d); // f(d) = P^-1(X) = sqrt(4d)
}
// non-uniform pdf
double pdf(double x) {
	return x / 2.0;
}
```

A linear PDF is probably a better approximation for a quadratic function than a uniform PDF
	-> we should try to make PDF match the integrand by tuning the PDF into a quadratic function.
$$p(r)=\begin{cases} 
0 \space\space\space\space\space\space\space\space\space\space\space\space r <0 \\
C\cdot r^2 \space\space\space\space\space0 \le r \le 2\\
0 \space\space\space\space\space\space\space\space\space\space\space\space 2 < r
\end{cases}$$
Integrating $p(r)$ from 0 to 2 should be 1 (i just skipped the integration)
Then, $C = \frac{3}{8}$

If we take an integral of PDF, we can get a CDF of the PDF.
$$p(r)=\begin{cases} 
0 \space\space\space\space\space\space\space\space\space\space\space\space r <0 \\
\frac{3}{8}\cdot r^2 \space\space\space\space\space0 \le r \le 2\\
0 \space\space\space\space\space\space\space\space\space\space\space\space 2 < r
\end{cases}$$
A CDF will be : $$P(r) = \frac{r^3}{8}$$
A $f(d)$ is inverse of CDF :
$$f(d) = 8d^{\frac{1}{3}}$$

With a single sample, it returns perfectly exact answer.

```cpp
double f(double d) { return 8.0 * pow(d, 1.0/3.0); } 
double pdf(double x) { return (3.0/8.0) * x*x; }

int main() {    
	int N = 1;    
	auto sum = 0.0;
    for (int i = 0; i < N; i++) {
        auto x = f(random_double()));
        sum += x*x / pdf(x);
    }
    std::cout << std::fixed << std::setprecision(12);
    std::cout << "I = " << sum / N << '\n';
}
```

In summary,
- less noise in the place where the PDF is big
- more noise where the PDF is small
If we choose a PDF that is higher in the place that have more noise and is lower in the place that have less noise, the total noise will be reduced.
-> we are steering our samples toward the parts of the **important** distribution.

Using a carefully chosen non-uniform PDF == importance sampling.






# 4. Monte Carlo Integration on the sphere of directions

In the last chapter, we learned :
- uniform random number
- more complicated ways of producing random numbers
- how to use them to generate random numbers of arbitrary distribution.

The topcis are all relate to one-dimensional cases, but in real world application, we encounter higher-dimensional problems. 
An important case of this in ray-tracing is producing random direction of ray.

Our original approach was to generate a random vector and reject it if it exceeds bounds of unit-sphere. (**rejection method.**)

Now we might have a PDF defined over two dimensions.
ex)
$$f(\theta, \phi) = cos^2(\theta)$$
Using monte carlo integration, we should sample $cos^2(\theta)/p(r)$ , where the p(r) == p(direction).
In terms of polar coordinates, it would be $p(\theta, \phi)$ whose integral is equal to one over the whole surface.

Let's take an easy case first.
We've already used `random_unit_vector` function to generate a unit direction in sphere a few times before.
Then what's the PDF of this function?
Because of this `random_unit_vector` is based on uniform random generator `random_double`, its pdf is $1/area = 1/4\pi$.

an example )
```cpp
double f(const Vec3& d) {
    auto cos_squared = d.z() * d.z();
    return cos_squared;
}

double pdf(const Vec3& d) {
    return 1 / (4 * pi);
}

int main() {
    int N = 1000000;
    auto sum = 0.0;
    for (int i = 0; i < N; i++) {
        auto d = random_unit_vector();
        auto f_d = f(d); // use a function f to randomly select samples from within this distribution [0,2]
//- If a sample `d` is more probable (i.e., has a higher pdf), dividing by that probability effectively reduces its contribution to the sum.
//- If a sample `d` is less probable (i.e., has a lower pdf), dividing by that probability increases its contribution to the sum.
        sum += f_d / pdf(d);
    }
    std::cout << std::fixed << std::setprecision(12);
    std::cout << "I = " << sum / N << '\n';
}
```

---

- the way to represent a single direction in 3D is the related point on the unit sphere
- About a range of directions, amount of area on the unit sphere that directions travel through.

We have radians in $\theta$ over one dimension, now we have *steradians(sr)* in $\theta, \phi$ over two dimensions, which is called 'Solid Angle'
Note that a sphere is a three-dimensional object, but its surface is only two-dimensional.

A Solid angle '$\omega$' can be visualized as below :
![](../../../../images/Pasted%20image%2020240225124743.png)
$$\omega=\frac{A}{r^2}$$, where $A$ is a projected area.

# 5. Light scattering
It's time to understand light transport equation.

## 5.1 albedo
First question : Is the light absorbed?
- probability of light being scattered : $A$
- probability of light being absorbed : $1-A$ 
, where $A$ means 'albedo' (whiteness)

Albedo is used to define some form of 'fractional reflectance'.
A fractional reflectance vary with
- color
- incident direction (the direction of incoming ray)

By using this fractional reflectance, we want to simulate the movement of photons through a space.
From basic Physics , we know the energy of a photon follows :
$$E = \frac{hv}{\lambda}$$, where $h$ is Planck constant, $v$ is frequency and  $\lambda$ is a wavelength.

Each individual photon has a tiny amount of energy, and **the absorption or scattering of a photon is probabilistically determined by the albedo of an object.**

Albedo can depend on color because some objects are more likely to absorb some wavelengths.

In most PBR renderers, we would use a predefined set of specific wavelengths for the light color (300nm, 350nm, 700nm and so on) than tristimulus RGB format.

Human visual system has 3 unique sets of color receptors called 'cones'.
Each of the cones are sensitive to different algebraic mixtures of wavelengths and the cones are referred to as long, medium and short cones.


## 5.2 Light scattering

Light scatters -> light has a directional distribution (this can be described as a PDF over solid angle)

From now on, scattering PDF == *pScatter()*.
- scattering PDF can vary with incident direction
	- so $pScatter(w_i, w_o)$
- Also, the pdf can vary with the wavelength of the light (ex. prism refracting white light into rainbow)
	- $pScatter(w_i, w_o, \lambda)$
- Finally, the pdf can depend on the scattering position.
	- $pScatter(X, w_i, w_o, \lambda)$, where $X=(x,y,z)$
- The albedo of an object can also depend on this quantities
	- $albedo = A(X,w_i, w_o, \lambda)$

The color of a surface is found by integrating these terms over the unit sphere by 'incident direction'.
$$Color_o(x, w_o, \lambda)=\int_{w_i}A(X,w_i, w_o, \lambda)\cdot pScatter(X,w_i, w_o, \lambda)\cdot Color_i(x, w_i,\lambda)$$
- albedo, pSacttering : works as filters to the light that is shining in the point.
This is a recursive function, so 

## 5.3 scattering PDF
We know that we can't make the 'i' of $w_i$ go to infinity in computations.
So here we can apply Monte carlo basic formula :
$$Color_o(x, w_o, \lambda)\approx \sum\frac{A(...)\cdot pScatter(...)\cdot Color_i(...)}{p(X, w_i, w_o, \lambda)}$$
, where $p$ is the PDF of whatever outgoing direction we randomly generate.

For a Lambertian surface, $pScatter()$ can be replaced with cosine density.
$$pScatter(...)=C\cdot cos(\theta_o)$$

And we need to integrate $pScatter$ over whole surface. (Note that we ignore the incident direction whose angle against normal is lower than zero.)
$$1= \int_{0}^{2\pi}\int_{0}^{\pi/2}C\cdot cos(\theta)dA$$

The '$dA$' term can be derived by following this image.
![](../../../../images/Pasted%20image%2020240229102649.png)
By Subsituiting $dA$ term with above equation, we can solve the integral of scattering PDF :
$$1= \int_{0}^{2\pi}\int_{0}^{\pi/2}C\cdot cos(\theta)sin(\theta)d\theta d\phi$$
Skipping the intermediate steps, The integral of $cos(\theta_o)$ is over the hemisphere is $\pi$.
$$1=C\cdot \pi$$
Finally, we need to normalize the integral by $C = \frac{1}{\pi}$.

In regard to Lambertian reflectance, the pdf only dependent on outgoing direction $w_o$. so we can simplify the parameter set as.
$$pScatter(X, w_i, w_o, \lambda) = pScatter(w_o) = \frac{\cos(\theta_o)}{\pi}$$

Let's assume that a pdf for randomly generated outgoing direction is equalt to $pScatter(w_o)$
$$p(w_o)=pScatter(w_o)=\frac{\cos(\theta_o)}{\pi}$$
Then we can simply outgoing color as :
$$Color_o(X, w_o, \lambda)\approx\sum A(...)\cdot Color_i(...)$$
Thanks to this assumption that two pdfs are same, this whole derivation is exactly same with a code we wrote before.
```cpp
return atteunation * ray_color(scattered, depth-1, world);
```

Standard reflection is defined by the *Bidirectional Reflectance Distribution Function*, shortly 'BRDF'
$$BRDF(w_i, w_o, \lambda)=\frac{A(...)\cdot pScatter(...)}{\cos(\theta_o)}=\frac{A(...)\cdot (cos(\theta_o)/\pi)}{\cos(\theta_o)}$$
Eventually The BRDF for a Lambertian surface :
$$BRDF = A/\pi$$


---



# 6. Apply Importance Sampling 

Now we want to send a bunch of extra rays toward right sources using a 
PDFs.
Let' assume some PDFs
- $pLight(w_o)$
- $pScatter(w_o)$
- $pSurface(w_o)$ - a PDF related to $pScatter(w_o)$

One of good things of PDF is that we can mix some of PDFs to form **mixture densities that are also PDFs**.

- ex)$$p(w_o)=\frac{1}{2}pSurface(w_o)+\frac{1}{2}pLight(w_o)$$
However, this fact is only kept when the weights of mixing equation are positive and add up to one. (in above example, weights are '$1/2, 1/2$' and they add up to one.)

Our mission is to figure out how to make the PDF larger where the product $pScatter(X,w_i, w_o)\cdot Color_i(X,w_i)$ is largest.
- For diffuse surfaces, By following the cosine law, the guessing where $Color_i$ is the largest is equivalent to guess where the most light is coming from.
- For mirror-like surfaces (nearly ideal specular reflection) , $pScatter$ matters more.

Now let's add scattering PDF in our code base and apply it into lambertian material.

```cpp
    double scattering_pdf(const ray& r_in, const hit_record& rec, const ray& scattered) const {
        auto cos_theta = dot(rec.normal, unit_vector(scattered.direction()));
        return cos_theta < 0 ? 0 : cos_theta/pi; // avoid pdf become zero
    }
```

```cpp
Color3 ray_color(const Ray& r, int depth, const Hittable& world) const {
	if (depth <= 0)
		return Color3(0, 0, 0);

	HitRecord rec;
	if (!world.hit(r, Interval(0.001, infinity), rec))
		return background;

	
	Ray scattered;
	Color3 atteunation;
	Color3 color_from_emission = rec.mat->emitted(rec.u, rec.v, rec.p);

	if (!rec.mat->scatter(r, rec, atteunation, scattered))
		return color_from_emission;

	double scattering_pdf = rec.mat->scattering_pdf(r, rec, scattered);
	double pdf = scattering_pdf; 
	Color3 color_from_scatter = (attenuation * scattering_pdf * ray_color(scattered, depth-1, world)) / pdf; // (A() * pScatter() * C) / pdf
	
	return color_from_emission + color_from_scatter;
}
```


## 6.3 Random Hemispherical sampling

If we take another scattering distribution that changes material property, we get slightly different result.( for example, more noisy image and inaccurate color on the surface)

Here's the cases :
- If we are not sure what the best sampling pattern is for our material, it may be a good choice to try uniform PDF.

In another perspective, this gives us an issue that we need to handle.
Because a bug in Monte carlo program is one of the most difficult thing to debug , we need some infrastructure to prevent it.

# 7. Generating random directions

## 7.1 random directions relative to the z-axis

- how to generate a random direction with inversion method?
	Let's assume that the normal of a surface is same with 'z-axis' and $\theta$ is the angle from normal.
	With a given Directional PDF on the sphere($p(w) = f(\theta)$), the 1-D PDFs on $\theta$ and $\phi$ are :
	$$a(\phi) = 1/2\pi$$
$$b(\theta)=2\pi f(\theta)\sin(\theta)$$
	Then, we can solve for the CDF of $\theta$ and $\phi$ from uniformly generated numbers $r_1, r_2$.
	- $$r_1 = \int_{0}^{\phi}a(\phi')d\phi'=\phi/2\pi$$
		Therefore, $\phi = r_1\cdot2\pi$ (a uniformly generated random number $r_1$ is ranged between 0 and 1, so by multiplying $2\pi$, we expand the result to cover full range of $\phi$)
	- $$r_2=\int_{0}^{\theta}b(\theta')d\theta'=2\pi f(\theta')sin(\theta')d\theta'$$
		 Let's assume that $f(\theta')=\frac{1}{4\pi}$, which is uniform density on the sphere.
		 Then ,
		 $$r_2=\frac{1-cos(\theta)}{2}, cos(\theta)=1-2r_2$$
	Ok, now we get the $\phi, cos(\theta)$ (we don't solve for $\theta$ because we just need to know cosine of it.)
	To generate a unit vector direction toward($\theta$, $\phi$), we convert to Cartesian coordinates :
	$$x=cos(\phi)\cdot sin(\theta), y=sin(\phi)\cdot sin(\theta), z=cos(\theta)$$
	Because we didn't solve for the $\theta$ , we can't know what the value of $sin(\theta)$ directly. But we can use this rule instead
	$$cos^2(\theta)+sin^2(\theta)=1$$

## 7.2 uniform sampling a Hemisphere

In the hemi-sphere case with uniform sampling, we can use $p(w)=f(\theta)=\frac{1}{2\pi}$.
Then, $$cos(\theta) = 1-r_2$$To solve for $f(\theta) = cos^3(\theta)$ , we just need to follow the monte carlo approach with PDF that we already did before.

## 7.3 cosine sampling a hemisphere
At this time, let's think about a PDF $$p(w)=f(\theta)=cos(\theta)/\pi$$, which is non-uniform.

By skipping all the steps to convert a generated direction against ($\theta, \phi$) into Cartesian coordinates,
We will get :
$$x = cos(2\pi r_1)\sqrt{r_2}, \space y=sin(2\pi r_1)\sqrt{r_2}, \space z = \sqrt{1-r_2}$$

To integrate samples,
1. create random cosine direction represented in Cartesian coordinates
2. By extracting 'z' component of the vector, compute $cos^3$ and its pdf

# 8. Orthonormal bases

From now on, we only took care of generating random directions relative to z-axis to make the concept simple.
However in real world application, we need to take care of this generation relative to surface normal.

## 8.1 Relative Coordinates.
Definition of Orthonormal basis 
	a collection of three mutually orthogonal unit vectors. This is a strict subtype of coordinate system.

The orthonoraml basis defines how distances and orientations are represented in the space.
But the orthonormal basis alone is not enough. To explain its displacement in the space, we always need Origin of the space.

Hence, we can represent a location like below :
$$Location = O' + u\text{u} + v\text{v} + w\text{w}$$, where $O'$ is origin in the coordinate system, ($u, v, w$)  coefficients are distance and ($\text{u},\text{v},\text{w}$) are basis.

## 8.2 Generate Orthonormal bases
We need to generate random directions with a set distribution relative to the surface normal vector.
For this, we only need direction because the direction is relative and has no specific origin.
We need two cotangent vectors that are perpendicular to surface normal and the two vectors are perpendicular to each other.
![](../../../../images/Pasted%20image%2020240304145724.png)

Suppose that any vector $\vec{a}$ (nonzero length, nonparallel to normal $\vec{n}$), Then we can generate cotangent vectors by using cross product.
$$\vec{S} = \vec{a}\times\vec{n}, \space\space \vec{s}=\frac{\vec{S}}{\lvert \vec{S}\rvert}$$
$$\vec{t}=\vec{s}\times\vec{n}$$ 
The problem is that we may not have an arbitrary vector $\vec{a}$.
In the case, we can pick an arbitrary axis and check to see if it's parallel to $\vec{n}$.

$\vec{n}$ is a normal of surface and it is unit-length vector at the same time by its definition.
So If one of component is close to -1 or 1 , that means the normal vector is nearly aligned with the axis corresponding to the component. 
```cpp
if(fabs(n.x()) > 0.9) // if absolute value of x component is greater than 0.9 , the normal nearly aligned with x-axis.
	a = vec3(0,1,0);
else
	a = vec3(1,0,0);
```
Then do cross products to get two cotangent vectors by following the equations above.

## 8.3 Update scatter function

- Lambertian material
```cpp
bool scatter(
    const ray& r_in, const hit_record& rec, color& alb, ray& scattered, double& pdf) const override {        
        OrthonormalBasis uvw;
        uvw.build_from_w(rec.normal); // build the bases by surface normal
        // get a direction depending on the basis.
        auto scatter_direction = uvw.local(random_cosine_direction());            scattered = ray(rec.p, unit_vector(scatter_direction), r_in.time());
        alb = albedo->value(rec.u, rec.v, rec.p);
        // Now dot product between surface normal and a direction represent by orthonormal basis.        
        pdf = dot(uvw.w(), scattered.direction()) / pi;        
        return true;
    }
```

# 9. Sampling Lights directly

As we already know, uniform sampling has a problem it may sample unimportant or any arbitrary direction.

Instead, we can use a PDF that sends more rays to the light (steering random directions toward light).

An easy way to pick a random direction toward light is that pick a random point on the light and compute a direction from origin to the random point.

For this, we need PDF $p(w)$ not to bias our render.

## 9.1 Getting the PDF of a light

With a given points $p$ and $q$, 
- the probability of sampling $dA$ : $p_q(q)\cdot dA$ 
- the probability of sampling differential area $dw$ on the sphere : $p(w)\cdot dw$ 
- geometric relationship between $dw$ and $dA$ (solid angle relationship)
	$$dw = \frac{dA\cdot cos(\theta)}{distance^2(p,q)}$$
	![](../../../../images/Pasted%20image%2020240305135332.png)
- the probability of sampling $dw$ and $dA$ must be the same, 
	$$p(w)dw = p_q(q)dA$$
	$p_q(q)$ is a uniform PDF , which is $1/A$
	Hence,
	$$p(w)\cdot \frac{dA\cdot cos(\theta)}{distance^2(p,q)}=\frac{dA}{A}$$
	$$p(w) = \frac{distance^2(p,q)}{cos(\theta)\cdot A}$$
This image shows the situation we take care right now.
![](../../../../images/Pasted%20image%2020240305131716.png)

## 9.2 Light sampling
Let's apply the above derivations in the application code.
```cpp
	ray scattered;
    color attenuation;        
    double pdf;        
    color color_from_emission = rec.mat->emitted(rec.u, rec.v, rec.p);
    
	if (!rec.mat->scatter(r, rec, attenuation, scattered, pdf))
		return color_from_emission;

	auto on_light = point3(random_double(213,343), 554, random_double(227,332)); // hard-coded random pointer on light
	auto to_light = on_light - rec.p; // a direction from hit point to light
	auto distance_squared = to_light.length_squared(); // length of the vector from p to q
	to_light = unit_vector(to_light); // normalize the direction vector.

    if (dot(to_light, rec.normal) < 0) // if dot product under horizon, don't care scattering
        return color_from_emission;

	// random value range is 213~343, 227~332. -> get a differential area
	double light_area = (343-213)*(332-227); 
	auto light_cosine = fabs(to_light.y());
	if (light_cosine < 0.000001)
		return color_from_emission;

	pdf = distance_squared / (light_cosine * light_area);
	scattered = ray(rec.p, to_light, r.time());

	double scattering_pdf = rec.mat->scattering_pdf(r, rec, scattered);
```

## 9.3 Swithcing to undirectional light
By setting back face color as black, we can reduce unneccessary noisy around light source.


# 10. Mixture densities

- Combine a pdf for $cos(\theta)$ and a pdf for sampling togeter.


## 10.1 abstractions for PDF
we have two pdf obviously :
- one for surface
- one for light

We've had a pdf that takes a direction and makes an output based on its distribution.
In general, If we can know the distribution of randomly generated directions, we should use a PDF with the same distribution for fastest convergence.

The pdf class will do
1. returning a random direction weighted by the internal PDF distribution
2. returning the corresponding PDF distribution value in that direction.

- cosine pdf
```cpp
class cosine_pdf {
  public:
// build Orthonormal basis based on the surface normal
    cosine_pdf(const vec3& w) { uvw.build_from_w(w); }

// return a value generated in same distribution.
    double value(const vec3& direction) const override {
        auto cosine_theta = dot(unit_vector(direction), uvw.w());
        return fmax(0, cosine_theta/pi);
    }
// generate a random cosine direction represented in local coordinate system
    vec3 generate() const override {
        return uvw.local(random_cosine_direction());
    }

  private:
    onb uvw;
};
```
usage in `ray_color`
```cpp
double pdf_val;
color color_from_emission = rec.mat->emitted(r, rec, rec.u, rec.v, rec.p);
if (!rec.mat->scatter(r, rec, attenuation, scattered, pdf_val))
    return color_from_emission;

	cosine_pdf surface_pdf(rec.normal);
	scattered = ray(rec.p, surface_pdf.generate(), r.time());
	pdf_val = surface_pdf.value(scattered.direction());
	double scattering_pdf = rec.mat->scattering_pdf(r, rec, scattered);

	// by following the color equation against incident ray and outgoing direction.
	color color_from_scatter = (attenuation * scattering_pdf * ray_color(scattered, depth-1, world)) / pdf_val;
```


- additional functions for hittable objects
	The implementation is following the exact equations being shown in section 9
```cpp
double pdf_value(const point3& origin, const vec3& v) const override {
	hit_record rec; 
	if (!this->hit(ray(origin, v), interval(0.001, infinity), rec))
		 return 0; 
	auto distance_squared = rec.t * rec.t * v.length_squared(); 
	auto cosine = fabs(dot(v, rec.normal) / v.length()); 
	return distance_squared / (cosine * area); 
} 
	
vec3 random(const point3& origin) const override { 
// uniformly generated random point on quadrilaterals.
	auto p = plane_origin + (random_double() * axis_A) + (random_double() * axis_B); 
	return p - origin; 
}
```

## 10.3 The mixture PDF class
We can create any linear mixture of any PDFs to form mixture densities that are also PDFs.
$$pMixture = w_1\cdot p_1 + w_2\cdot p_2 + ... + w_n\cdot p_n$$
$$w_1 + w_2 + ... + w_n = 1$$


# 11. architectural decisions

In this series, the author used mixture-density approach instead of shadow rays.

- traditional shadow rays approach
	- still can see in most professional path tracers (with predefined number of shadow rays)
	- send terminal shadow rays to random lights to **determine** if the intersecion is lit by random lights or completely in shadow.
		-> After the rays terminate either at a light or at an occluding surface, the initial path tracing ray continues on
	- more cheaper than mixture-density and is becoming incresingly common in realtime.
	
- mixture-density approach
	- rays that check for unobstructed path from an intersection point to a given light source
	- can sample windows or bright parts under door that we may think important.

Also, we need to remove hard-coded PDF construction in `ray_color`.
- we can skip $f()/p()$ by making specular reflection 
- or set surface roughness to a very small(but nonzero) -> almost mirror-like reflection

Additionally, if we want to add an environment maps, we need to normalize RGB to floating point between 0 and 1 which is represented in 0~255 bytes in current implementation.

Finally, a more physically based renderer requires a color format as spectral colors and maybe polarization rather than simple RGB. (definitely more difficult to implement.)

# 12. cleaning up PDF management

We've only had so far :
- pdf related to the shape of the light
- pdf related to the normal vector and type of surface

now we need to add :
- information about the light (to ask `material` function for a PDF)
- information about if the scattered ray is specular 

## 12.1 diffuse vs specular

There is a kind of material that has two different material properties : partially ideal specular and partially diffuse.

For simplicity, we can have the material randomly decide whether it is diffuse or specular.
- we need to be careful when we ask for the PDF value based on the choice among diffuse or specular surface.
- Then, let `ray_color` function know whether the ray is diffuse or specular.

Let's add new record abstraction : `scatter_record`
```cpp
class scatter_record {
public:
	Color atteunation;
	shared_ptr<pdf> pdf_ptr;
	bool skip_pdf;
	Ray skip_pdf_ray;
}
```

By using this record abstraction, we can make the lambertial scattering simpler
```cpp
bool scatter(const ray& r_in, const hit_record& rec, scatter_record& srec) const override { 
// update scatter record.
	srec.attenuation = albedo->value(rec.u, rec.v, rec.p); 
	srec.pdf_ptr = make_shared<cosine_pdf>(rec.normal); 
	srec.skip_pdf = false; 
	return true; 
}
```

For more detailed code, refer to the book.

Then we can modify specular scattering :
```cpp
bool scatter(const ray& r_in, const hit_record& rec, scatter_record& srec) const override { 
	srec.attenuation = albedo; 
	srec.pdf_ptr = nullptr; 
	srec.skip_pdf = true; 
	vec3 reflected = reflect(unit_vector(r_in.direction()), rec.normal);
	srec.skip_pdf_ray = ray(rec.p, reflected + fuzz*random_in_unit_sphere(), r_in.time()); 
	return true; 
}
```


## 12.3 sampling a sphere object

For the metal-like material, we are just skipping the PDF right now.
But we would also want a PDF to reduce noise created by reflections on metal material.

Because it is quite diffcult to make a PDF for a block, we would implement it for sphere just for now.





# Reference
 - This markdown was writtien entirely based on :
	[_Ray Tracing: The Rest of Your Life](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html)
- differential unit area equation image
	[pbr-book.4ed](https://www.pbr-book.org/4ed/Radiometry,_Spectra,_and_Color/Working_with_Radiometric_Integrals)