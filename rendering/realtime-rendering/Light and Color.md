
# Radiometry

- deals with the measurement of electromagnetic radiation (radiation propagates such as waves)
- Each electromagnetic waves with different wavelengths(distance between same phases) tends to have different properties. 

## Terms
- radiant flux : basic unit of Radiametry, **flow of radiant energy over time** , measure in $W(watts)$
- irradiance : density of radiant flux with respect area. $d\Phi/dA$ 
  The area can be imaginary.
- solid angle : 
  An angle can be thought of a measure of the size of a continuous set of directions in a plane.
  a value in radians equal to the the length of the arc (a circle has radius 1).
  Similarily, In three-dimensional space, the set of directions is measured in steradians, which is a circular patch on the unit sphere.
  **$4\pi$ steradians covers the whole unit sphere**
- radiant intensity : density of radiant flux with respect to solid angle.
- radiance : radiant flux with respect to both area and solid angle.
  This quantities is the thing measured by sensors(such as camera, human eye)

In Summary,

| Name              | Symbol | Units                                            |
| ----------------- | ------ | ------------------------------------------------ |
| radiant flux      | $\Phi$ | watts($W$)                                       |
| irradiance        | $E$    | $W/m^2$                                          |
| radiant intensity | $I$    | $W/sr$, where sr is abbreviation of 'steradians' |
| radiance          | $L$    | $W/(m^2sr)$                                      |

A radiance distribution is composed of five variables 
- x,y,z for position
- $w_i, w_o$ for directions

With this function, we can describe all light traveling anywhere in space.
An important property of radiance is that it is constant regarding to distance from viewer. (Further object from viewer takes fewer pixels, but radiance doesn't change.)


---
# Photometry

Photometry is similar to radiometry, but the key difference is that it weights everything by the sensitivity of the human eye.

To convert photemetric units into radiometric units, we just need to multiply the units by the *CIE photometric curve*.

![](../../images/Pasted%20image%2020240805161444.png)

Except for the conversion curve and units of measurement, there is no difference between photometry and radiometry.

# Colorimetry

Colorimetry is also strongly related to the light's spectral power distribution (but not one to one relationship)

- it deals with the relationship between **spectral power distributions** and the **perception of color**.

Human perceives a color using three types of cone receptors in the retina.
Each of receptors responds in a different way to various wavelengths.

From color matching experiment, we got a function to convert spectral power distribution to three values.

An example of color matching functions are denoted below :
$$X = \int_{380}^{708}s(\lambda)\bar{x}(\lambda)d\lambda,\space Y = \int_{380}^{708}s(\lambda)\bar{y}(\lambda)d\lambda, \space Z = \int_{380}^{708}s(\lambda)\bar{z}(\lambda)d\lambda$$
And the '$X,Y,Z$' are weights that define a color in CIE XYZ space.


Sometimes it is convenient to separate colors into two parts : luminance(brightness) and chromaticity.
Chromaticity is a character of a color which is independent to luminance.

The chromaticity diagram describes a plane and it is important to understand how color is used in rendering and which limiations the rendering system have.

Display device(ex. monitor) controls *display primaries*.
The display primaries emit lights with a particular spectrol power distribution.
With given color, each of primaries is scaled and added up to a single distribution that the viewer perceives.

In below picture, there is a white-colored triangle. The three corners of the triangle are corresponding to each primaries in display device. (mostly saturated red, green and blue.)
![](../../images/Pasted%20image%2020240805165714.png)

## CIE 1976 UCS Diagram

![](../../images/Pasted%20image%2020240805171334.png)
- sRGB is the most commonly used in real-time rendering field.
- Most of computer monotiors adopted sRGB color space, but there are some devices that applies broader color space such as ACEScg, DCI-P3.
- conversion from a RGB space to XYZ space is linear (done by matrix operation)



# Rendering with RGB colors

RGB values represent perceptual quantities. Therefore, it is not right to use it for physically based rendering.
The accurate method is to perform all rendering computations on spectral quantities, then convert it into RGB colors at display time.  

But In interactive applications, most of them are using RGB rendering rather than spectral rendering. (The errors caused by using RGB rendering is somewhat subtle in real-time application.)







# Reference
- Realtime Rendering 4th.ed