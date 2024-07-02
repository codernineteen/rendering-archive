
- continuous time fourier transform
	$$CTsignal \Leftrightarrow X(j\omega)$$
	, where $X(jw)$ is continous frequency domain function.
- continuous time fourier transform
$$DTsignal \Leftrightarrow X(e^{j\omega})$$
	, where $X(jw)$ is continous frequency domain function.

On the other hand, the Discrete Fourier transform makes discrete frequency domain as a result of transformation.
$$DTsignal \Leftrightarrow Discrete\space Frequency$$

In other words, if we give DFT system N inputs, it will return N ouputs as frequencies.

## Definition of DFT

After taking a DTFT of $x[n]$ , where $x[n]$ has values from 0 to $N-1$, Take values from the result function of DTFT for every $k\cdot\frac{2\pi}{N}$ intervals($k=0..N-1$).
Therefore,
1. $x[n] \Leftrightarrow X(e^{j\omega})$
2. Take values from $X(\Omega)$ for every intervals and make $X_k = X(k\frac{2\pi}{N})$ 

### 1. $x[n] \Leftrightarrow X(e^{j\omega})$
$$X(\omega) = \sum_{<N>}x[n]\cdot e^{-j\omega n}$$

### 2. $X(e^{j\omega})$ to DFT 
$$X_k = X(k\frac{2\pi}{N})=\sum_{n=0}^{N-1}x[n]\cdot e^{-jk\frac{2\pi}{N}n}$$ and It actually equivalent to Discrete Fourier series expression :
$$\sum_{n=0}^{N-1}x[n]\cdot e^{-jk\frac{2\pi}{N}n} = a_k\cdot N$$, where $a_k=\frac{1}{N}\sum_{n=0}^{N-1}x[n]\cdot e^{-jk\frac{2\pi}{N}n}$

---
## DFT matrix

When we carefully into the DFT equation, we can know that we do multiplication by alternating a variable 'k'. 
Hence, we can represent the equation as multiplication of matrices.

Let's assume that we only care about the case where k is 0,1 and 2.
$$\begin{equation}
\begin{bmatrix}X_1 \\ X_2\\ X_3\end{bmatrix}=
\begin{bmatrix}
e^{-j0\frac{2\pi}{N}0} && e^{-j0\frac{2\pi}{N}1} && e^{-j0\frac{2\pi}{N}2} \\
e^{-j1\frac{2\pi}{N}0} && e^{-j1\frac{2\pi}{N}1} && e^{-j1\frac{2\pi}{N}2} \\ 
e^{-j2\frac{2\pi}{N}0} && e^{-j2\frac{2\pi}{N}1} && e^{-j2\frac{2\pi}{N}2}
\end{bmatrix}
\cdot
\begin{bmatrix}x[0] \\ x[1]\\ x[2]\end{bmatrix}
\end{equation}$$

However, we can't call it above matrix as DFT matrix yet.
First we need to make the matrix orthogonal (It is called as unitary matrix with complex number components.)

While each columns of the matrix is perpendicular to others, the magnitude of them is not equal to zero.
To achieve it, we just need to multiply $1/\sqrt{N}$ to every components so that the magnitude of each columns would be equal to 1.
$$\begin{bmatrix}
\frac{1}{\sqrt{N}}e^{-j0\frac{2\pi}{N}0} && \frac{1}{\sqrt{N}}e^{-j0\frac{2\pi}{N}1} && \frac{1}{\sqrt{N}}e^{-j0\frac{2\pi}{N}2} \\
\frac{1}{\sqrt{N}}e^{-j1\frac{2\pi}{N}0} && \frac{1}{\sqrt{N}}e^{-j1\frac{2\pi}{N}1} && \frac{1}{\sqrt{N}}e^{-j1\frac{2\pi}{N}2} \\ 
\frac{1}{\sqrt{N}}e^{-j2\frac{2\pi}{N}0} && \frac{1}{\sqrt{N}}e^{-j2\frac{2\pi}{N}1} && \frac{1}{\sqrt{N}}e^{-j2\frac{2\pi}{N}2}
\end{bmatrix}$$, where $N$ is 3 in the example.

Why we need to represent the matrix in a unitary form?
It is because we can utilize same property of orthogonal matrix with real number in unitary matrix by doing so.
- When $D$ is orthogonal matrix,
$$D^TD = I$$
- When D is unitary matrix,
$$(D^*)^TD=I$$
Therefore $(D^*)T$ is equivalent to the inverse of matrix $D$. 
Without any need to do gauss-elimination or $LU$ decomposition to get a inverse matrix, we just need to take transpose of original matrix to get the inverse matrix.
Very Easy!

Now it is really easy to derive the signal $x[n]$ by multiplying the inverse matrix to both sides of the equation.
$$X=Dx$$
$$D^{-1}X = D^{-1}Dx = Ix = x$$

Finally, DFT equation will be slightly modifed into new form :
$$X_k = \frac{1}{\sqrt{N}}\sum_{n=0}^{N-1}x[n]\cdot e^{-jk\frac{2\pi}{N}n}$$

---

## Cyclic convolution

We know that the convolution in time domain establishes multiplication between results of fourier transform of two signal in frequency domain.

However, In DFT, we have slight different feature compared to general convolution.
Shortly, it is cyclic convolution of two periodic signals.
$$x[n]\circledast h[n] \Leftrightarrow X_kH_k$$
We could show the DFT analysis equation has same form with Discrete time Fourier Series(DTFS) and DTFS represents an aribtrary signal as a sum of periodic complex exponential functions.

In summary, 'Cyclic convolution' means 'convolution of periodic signals'.
Then how we do that?

we can do convolution of two periodic signals by accumulating only inputs in a single period.

Originally the convolution equation is represented as :
$$\int_{-\infty}^{\infty}x(\tau)h(t-\tau)d\tau$$

But as i mentioned, now we only care the convolution in a single period:
$$\int_{T_0}x(\tau)h(t-\tau)d\tau$$