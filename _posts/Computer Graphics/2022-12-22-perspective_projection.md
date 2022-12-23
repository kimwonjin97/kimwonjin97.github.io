---
title:  "Perspective Projection & Distance Comparison"
excerpt: "Perspective Projection & Distance Comparison"

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

## Perspective Projection
Previously for orthographic projection we assumed that the ray direction are (0.0f, 0.0f, 1.0f) regardless of the pixel location. Thus the information about how far each object is, are difficult to identify from the resultant visual content.

To account for the information about the distance instead of each ray casting per pixel in the same z direction, we need these rays to be in the direction dependent on the eye position(perpective). 

<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-22-14-13-19.png"><img src="/assets/images/posts/graphics/2022-12-22-14-13-19.png"></a>
</figure>


```c++
const auto rayDir = glm::normalize(pixelPosWorld - eyePos);
```

## Distance Comparison
Now we apply the perspective projection to our original code. Then are we able to depict the scene as similar to reality? 
The Potential output would look something like the followings.

<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-22-14-17-41.png"><img src="/assets/images/posts/graphics/2022-12-22-14-17-41.png"></a>
</figure>

Even though blue sphere is behind the green sphere and green sphere is behind the red sphere, the results seems to have drawn the overlapping region in incorrect order. To solve this, if there is any overlapping region, one that is closer needs to be drawn. 
```c++
Hit FindClosestCollision(Ray& ray)
{
    float closestD = 1000.0; 
    Hit closest_hit = Hit{ -1.0, dvec3(0.0), dvec3(0.0) };

    for (int l = 0; l < objects.size(); l++)
    {
        auto hit = objects[l]->CheckRayCollision(ray);

        if (hit.d >= 0.0f)
        {
            if (hit.d < closestD)
            {
                closestD = hit.d;
                closest_hit = hit;
                closest_hit.obj = objects[l];
            }
            //hit.obj = objects[l];
            //return hit;
        }
    }

    return closest_hit;
}
```

final result would look like this.

<figure class="full">
    <a href="/assets/images/posts/graphics/2022-12-22-14-22-25.png"><img src="/assets/images/posts/graphics/2022-12-22-14-22-25.png"></a>
</figure>