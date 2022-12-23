---
title:  "Bloom effect"
excerpt: "Using 2D arrays as if it was 1D array"

categories:
  - Computer Graphics
tags:
  - [cpp]

toc: true
toc_sticky: true

date: 2022-12-20
last_modified_at: 2022-12-20
mathjax: true
---

Before we start, 
1. std::clamp(c++17) in `algorithm` header is useful when deal with 2D image


assume this following function is inside update function.

```c++
for(int j=0; j<image.height; ++j)
{
  for(int i=0; i<iamge.width; ++i)
  {
    img.pixels[i+image.width*j].v[0] = std::clamp(image.pixels[i+image.width*j]*1.01f, 0.0f, 1.0f);
    img.pixels[i+image.width*j].v[1] = std::clamp(image.pixels[i+image.width*j]*1.01f, 0.0f, 1.0f);
    img.pixels[i+image.width*j].v[2] = std::clamp(image.pixels[i+image.width*j]*1.01f, 0.0f, 1.0f);
  }
}
```

following code multiplies a small value to their RGB making the image brighter. We expect that image to become completely black however, some of the pixels will stay in yellow or red. This is because some of the pixels were originally 0 and multiplying wasn't affecting them. To resolve this when we read the image we add very small values to each pixel. (ex. `image.pixels[i].v[0] += 1e-2f;`)


## Kernel

Kernel(Convolution matrix) is a small matrix used for sharpening, blurring, embossing, edge detection, and more. 

some of the known kernel could be found in this [wikipedia](https://en.wikipedia.org/wiki/Kernel_(image_processing)) page.

## Convolution
To apply kernel to the entire image we have to use a mathematical operation called convolution. 

<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-21-12-28-34.png"><img src="/assets/images/posts/graphics/2022-12-21-12-28-34.png"></a>
    <figcaption>reference: https://medium.com/@bdhuma/6-basic-things-to-know-about-convolution-daef5e1bc411</figcaption>
</figure>

### Seperable convolution
2D filter could be separated into two 1D filters if the 2D filter could be expressed as an outer product of 2 1D filter. We prefer two 1D filters because it reduces the number of computations. 

How to know if the kernel is separable? -> calculate the rank of the matrix(seperable if rank is 1).


## Box and Gaussian blur

### box blur
```c++
    //consider kernel 1/5[1,1,1,1,1].


    //copy created
    std::vector<Vec4> pixelsBuffer(this->pixels.size()); 

	for (int j = 0; j < this->height; j++)
	{
		for (int i = 0; i < this->width; i++)
		{
            //temporary sum
			Vec4 neighborColorSum{ 0.0f, 0.0f, 0.0f, 1.0f };
			for (int si = 0; si < 5; ++si)
			{
                Vec4 neighborColor = is_horizontal_blur? this->GetPixel(i + si - 2, j) : this->GetPixel(i, j + si -2);    
                neighborColorSum.v[0] += neighborColor.v[0];
				neighborColorSum.v[1] += neighborColor.v[1];
				neighborColorSum.v[2] += neighborColor.v[2];
			}
			pixelsBuffer[i + this->width * j].v[0] = neighborColorSum.v[0] * 0.2f; 
			pixelsBuffer[i + this->width * j].v[1] = neighborColorSum.v[1] * 0.2f;
			pixelsBuffer[i + this->width * j].v[2] = neighborColorSum.v[2] * 0.2f;

		}
	}
```

### Gaussian blur
```c++
    std::vector<Vec4> pixelsBuffer(this->pixels.size());

	//reference: https://followtutorials.com/2013/03/gaussian-blurring-using-separable-kernel-in-c.html
	const float weights[5] = { 0.0545f, 0.2442f, 0.4026f, 0.2442f, 0.0545f };

	for (int j = 0; j < this->height; j++)
	{
		for (int i = 0; i < this->width; i++)
		{

			Vec4 neighborColorSum{ 0.0f, 0.0f, 0.0f, 1.0f };
			for (int si = 0; si < 5; ++si)
			{
				Vec4 neighborColor = is_horizontal_blur? this->GetPixel(i + si - 2, j) : this->GetPixel(i, j + si -2);
				neighborColorSum.v[0] += neighborColor.v[0] * weights[si];
				neighborColorSum.v[1] += neighborColor.v[1] * weights[si];
				neighborColorSum.v[2] += neighborColor.v[2] * weights[si];
			}
			pixelsBuffer[i + this->width * j].v[0] = neighborColorSum.v[0];
			pixelsBuffer[i + this->width * j].v[1] = neighborColorSum.v[1];
			pixelsBuffer[i + this->width * j].v[2] = neighborColorSum.v[2];
		}
	}
```
reference
## Bloom
1. leave only the bright part of the image (make RGB to 0 if they are below a certain threshold)
2. apply gaussian blur to the image gotten from step 1
3. add the orignal image with the image gotten from step 2.

``` c++
void Image::Bloom(const float& th, const int& numRepeat, const float& weight)
{
	const std::vector<Vec4> pixelsBackup = this->pixels;
	
	//reference for relative luminance(Y = 0.2126*R + 0.7152*G + 0.0722*B): https://en.wikipedia.org/wiki/Relative_luminance
    // 1. leave bright part of the image
	for (int j = 0; j < height; j ++)
		for (int i = 0; i < width; i++)
		{
			auto& c = this->GetPixel(i, j);
			const float relativeLuminance = c.v[0] * 0.2126f + c.v[1] * 0.7152f + c.v[2] * 0.0722f;

			if (relativeLuminance < th)
			{
				c.v[0] = 0.0f;
				c.v[1] = 0.0f;
				c.v[2] = 0.0f;
			}
		}

	// apply blur to image gotten from step 1
	for (int i = 0; i < numRepeat; i++)
	{
		this->GaussianBlur5();
	}

    // add orignal image with the image gotten from step2(apply weight to decide intensity of the bloom effect).
	for (int i = 0; i < pixelsBackup.size(); i++)
	{
		this->pixels[i].v[0] = std::clamp(pixels[i].v[0] * weight + pixelsBackup[i].v[0], 0.0f, 1.0f);
		this->pixels[i].v[1] = std::clamp(pixels[i].v[1] * weight + pixelsBackup[i].v[1], 0.0f, 1.0f);
		this->pixels[i].v[2] = std::clamp(pixels[i].v[2] * weight + pixelsBackup[i].v[2], 0.0f, 1.0f);

	}
}
```