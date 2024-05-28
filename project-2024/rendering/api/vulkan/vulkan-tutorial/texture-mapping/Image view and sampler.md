
Two resources for graphics pipeline to sample an image
- texture image view 
	images accessed through image views rathen than directly.
	Creating an image view is based on the steps we've already treated in swap chain creation sector.
- sampler
	 textures are usually accessed through samplers.
	 Sampler does filtering and tranformations to compute the final color that is retrieved
	- when number of fragments mapped to a texel is larger than one, the image can be shown without filtering
	  ![](../../../../../../images/Pasted%20image%2020240126095539.png)
	  But If takes 4-neareset texels and interpolates them bilinearly, we can get a smoother image.
	  ![](../../../../../../images/Pasted%20image%2020240126095633.png)
	  A Sampler automatically applies this filtering for us when reading a color from the texture.
	  - In the opposite way, if we have more texels than fragments  
		With anistropic filtering, you can see that the texture at a sharp angle is more clear
	![](../../../../../../images/Pasted%20image%2020240126100142.png)
	And about transformations, a sampler can do this kind of things (related to addressing mode)
	![](../../../../../../images/Pasted%20image%2020240126100626.png)


To create a sampler, use below struct
`VkSamplerCreateInfo`
- sType
how to interpolate texels
- magFilter 
- minFilter
specify addressing mode per axis (for texture space)
- addressModeU
- addressModeV
- addressModeW
	- `VK_SAMPLER_ADDRESS_MODE_REPEAT`: Repeat the texture when going beyond the image dimensions.
	- `VK_SAMPLER_ADDRESS_MODE_MIRRORED_REPEAT`: Like repeat, but inverts the coordinates to mirror the image when going beyond the dimensions.
	- `VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_EDGE`: Take the color of the edge closest to the coordinate beyond the image dimensions.
	- `VK_SAMPLER_ADDRESS_MODE_MIRROR_CLAMP_TO_EDGE`: Like clamp to edge, but instead uses the edge opposite to the closest edge.
	- `VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_BORDER`: Return a solid color when sampling beyond the dimensions of the image.
Options for anistropic filtering
- anistropyEnable
- maxAnistropy - amount limits of texel samples that can be used to calculate the final color. We need to query physical device properties first to map the information with this field.
- borderColor - about which color is returned when sampling beyond the image with clamp to border addressing mode.
- unnormalizedCoordinates - coordinate system for texels.
	-  without normalization -> 0~tex width, 0 ~ tex height
	- with normalization -> 0 ~ 1, 0 ~ 1
- compareEnable - mainly used for PCF(texels will first be compared to a value)
- compareOp
- mipmapMode, mipLodBias, minLod, maxLod