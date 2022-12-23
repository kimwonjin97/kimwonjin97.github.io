---
title:  "lighting effect"
excerpt: "using Phong reflection model"

categories:
  - Computer Graphics
tags:
  - [cpp]

toc: true
toc_sticky: true

date: 2022-12-22
last_modified_at: 2022-12-22
mathjax: true
---

In this post we are going to look at how to incorporate lighting to make object more realistic. This will be achieved by looking at old but still powerful model called "Phong reflection model" [wikipedia page](https://en.wikipedia.org/wiki/Phong_reflection_model).

Phong model describes the way a surface reflects light and its by combination of diffuse reflection of rough surfaces with specular reflection of shiny surfaces. The image from wikipedia does the good job of discribing this.

<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-22-12-03-51.png"><img src="/assets/images/posts/graphics/2022-12-22-12-03-51.png"></a>
</figure>


# Diffuse

<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-22-12-06-57.png"><img src="/assets/images/posts/graphics/2022-12-22-12-06-57.png"></a>
</figure>



We will use the diagram above to talk about how diffusion is calculated. Consider you have two light source \\(l_1\\) and \\(l_2\\). The light source that has a more similar unit direction with the normal to the surface(\\(l_1\\)) has a stronger effect than the other(\\(l_2\\)). However,  one light source that forms more than 90 degrees with the surface normal(beyond the horizon) would not affect the light at all. We could formulate this behavior into the equation(right bottom corner of the diagram). 

code for the diffusion part is:

```c++
const vec3 dirToLight = glm::normalize(light.pos - hit.point);
const float diff = glm::max(glm::dot(hit.normal, dirToLight), 0.0f);
```


# Specular
In diffuse surface, light incoming from the light source encounters rough surface and scatters everywhere(ex. wood, paper). In oppose to diffuse surface, the specular surface perfectly reflects the incoming light in a expected angle. 
<figure class="half">
    <a href="/assets/images/posts/graphics/2022-12-22-12-08-04.png"><img src="/assets/images/posts/graphics/2022-12-22-12-08-04.png"></a>
    <a href="/assets/images/posts/graphics/2022-12-22-12-08-17.png"><img src="/assets/images/posts/graphics/2022-12-22-12-08-17.png"></a>
</figure>


The reflected ray are responsible for the intensity of light that we could percieve. Intensity also depends on the angle of our sight and the reflected ray. Thus formula for specular is derived as shown in diagram.

```c++
const vec3 reflectDir = 2.0f * dot(hit.normal, dirToLight) * hit.normal - dirToLight; // r = 2 (n dot l) n - l
const float specular = glm::pow(glm::max(glm::dot(-ray.dir, reflectDir), 0.0f), sphere->alpha); //glm::pow depends on the material that we use. 
```


# Result
<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-22-12-33-19.png"><img src="/assets/images/posts/graphics/2022-12-22-12-33-19.png"></a>
</figure>

