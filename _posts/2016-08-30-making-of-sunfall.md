---
title: "Sunfall"
excerpt: "How I implemented simple crepuscular rays and volumetric dust"
date: 2016-08-30
categories: blog
tags: [shadertoy]
header:
  teaser: assets/blog/sunfall/teaser.jpg
---

[Sunfall](https://www.shadertoy.com/view/4ltGDS) is a shadertoy that I developed to try a simple crepuscular ray effect using *radial blur*. This post briefly goes through some of the steps in my implementation.

<figure>
  <iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/4ltGDS?gui=false&t=10&paused=false&muted=false" allowfullscreen></iframe>
</figure>

Sky and Terrain
---------------

I started by coloring the sky with a mix of blue and orange according to approximate the look of a sunset. I raymarched the terrain using Ridged [Perlin Noise](https://en.wikipedia.org/wiki/Perlin_noise) and composited the result with the background by applying a simple fog model. The code below implements the terrain. Notice that I use a low-quality version in the raymarcher and a high-quality one for computing the normals, which speeds up the shader significantly.

<figure class="half">
  <img src="/assets/blog/sunfall/sunfall-2.jpg" title="Background Sky">
  <img src="/assets/blog/sunfall/sunfall-4.jpg" title="Terrain after compositing">
  <figcaption>Background sky and terrain before and after compositing</figcaption>
</figure>

{% highlight glsl %}
// Perlin Noise
float noise(vec2 uv) {    
    vec2 iuv = floor(uv); // grid ID
    vec2 fuv = fract(uv); // position inside grid

    // compute all four corners in one vec4
    vec4 i = vec4(iuv, iuv + 1.0);

    // compute the vector from the corners to fuv in one vec4
    vec4 f = vec4(fuv, fuv - 1.0); 

    // transform the grid corners onto texel coords
    i = (i + 0.5) / iChannelResolution[0].xyxy;

    // sample the random texture
    // (must use nearest filter and no mipmaps!)
    vec2 grad_a = 2.0 * texture2D(iChannel0, i.xy).rg - 1.0;
    vec2 grad_b = 2.0 * texture2D(iChannel0, i.zy).rg - 1.0;
    vec2 grad_c = 2.0 * texture2D(iChannel0, i.xw).rg - 1.0;
    vec2 grad_d = 2.0 * texture2D(iChannel0, i.zw).rg - 1.0;

    // gradient dot products
    float a = dot(f.xy, grad_a);
    float b = dot(f.zy, grad_b);
    float c = dot(f.xw, grad_c);
    float d = dot(f.zw, grad_d);

    // quintic interpolation
    fuv = fuv*fuv*fuv*(fuv*(fuv*6.0 - 15.0) + 10.0);    
    return mix(mix(a, b, fuv.x), mix(c, d, fuv.x), fuv.y);
}

// Low quality, turbulent FBM (for raymarching)
float fbm(vec2 uv) {
    float h = 0.0, a = 1.0;    
    for (int i = 0; i < 4; ++i) {
        h += 1.0-abs(a * noise(uv)); // ridged perlin noise
        a *= 0.45; uv *= 2.02;
    }        
    return h;
}

// High quality, turbulent FBM (for computing normals)
float fbmH(vec2 uv) {
    float h = 0.0, a = 1.0;    
    for (int i = 0; i < 9; ++i) {
        h += 1.0-abs(a * noise(uv)); // ridged perlin noise
        a *= 0.45; uv *= 2.02;
    }        
    return h;
}
{% endhighlight %}

I computed normal vectors using [central differences](https://en.wikipedia.org/wiki/Finite_difference) over the terrain distance function. The offset at which the terrain is sampled increases quadratically with the distance to the camera, so that details are filtered out to avoid aliasing.

{% include figure image_path="/assets/blog/sunfall/normals.gif" caption="Filtering normals as a function of distance to the camera" %}

Terrain Shading
---------------

I implemented a diffuse (Lambertian) and specular (Sloan-Hoffman) [BRDF](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) models for shading the terrain. A fill light is also present at the camera position, but it only contributes diffuse lighting. A constant ambient illumination model is applied to wash out the whole scene.

{% highlight glsl %}
vec3 shade(vec3 ro, vec3 rd, float t) {    
    const float pi = 3.141592;
    
    vec3 p = ro + t * rd;
    vec3 n = normal(p, t);    
    
    // Diffuse        
    vec3 diff_brdf = g_sand_diff / pi;
    
    // Specular
    float m = 20.0;
    vec3 h = normalize(g_sundir - rd);
    vec3 spec_brdf = vec3((m + 8.0)*pow(max(dot(n, h), 0.0), m)/(8.0*pi));    
    float schlick = 0.045 + 0.955*pow(1.0 - dot(h, -rd), 5.0);
    
    // Rendering Equation
    vec3 brdf = mix(diff_brdf, spec_brdf, schlick);    
    vec3 col = brdf * g_suncol * max(dot(n, g_sundir), 0.0);
	        
    // Fill light hack
    col += 0.75*g_sky_blue*diff_brdf*max(dot(n, -rd), 0.0);
    
    // Ambient Hack
    m = smoothstep(0.0, 1.0, n.y);
    col = mix(col, g_sand_diff * vec3(0.382, 0.39, 0.336)*m, 0.2);
    
    float fog = exp(-0.015*t);
    return mix(bg(rd), 7.0*col, fog);
}
{% endhighlight %}

{% include figure image_path="/assets/blog/sunfall/sunfall-7.jpg" caption="Sun light, fill light, and ambient light composition" %}

Sun Rendering and Compositing
-----------------------------

In a first pass, I created an occlusion mask that specifies which pixels receive light directly from the sun. A second pass applies a radial blur onto the mask so as to cheaply simulate crepuscular rays. The result is composited with the sky and terrain using a yellow color.

<figure class="half">
  <img src="/assets/blog/sunfall/sunfall-9.jpg">
  <img src="/assets/blog/sunfall/sunfall-10.jpg">
  <figcaption>Occlusion mask of the sun and the result of radial blur</figcaption>
</figure>

{% include figure image_path="/assets/blog/sunfall/sunfall-11.jpg" caption="Composition of the sun, terrain, and background" %}

Volumetric Dust
---------------

Finally, I included a dust effect by raymarching a 3D [Value Noise](https://en.wikipedia.org/wiki/Value_noise) procedural function. After hitting the terrain, the raymarcher starts walking backwards at fixed steps to render the dust. For shading, instead of computing the normals and taking the dot product with the sun direction, I used [directional derivatives](https://en.wikipedia.org/wiki/Directional_derivative) to speed up this process, as explained in [this link](http://www.iquilezles.org/www/articles/derivative/derivative.htm).

{% highlight glsl %}
vec3 volmarch(vec3 ro, vec3 rd, float t, vec3 col) {    
    float pix = 2.0 / iResolution.y; // pixel size

    // raymarch backwards after finding the intersection t-value
    // only take 10 samples in the 3D volume for efficiency
    for (int i = 0; i < 10; ++i) {
    
        // add some translational movement in the point being shaded
        // this simulates a cheap wind effect...
        vec3 p = ro + t * rd - vec3(-.5, .5, .1)*iGlobalTime;
    	float EPS = pix * t;
        
        float f1 = 0.25*fbm(p);
        float f2 = 0.25*fbm(p + EPS*g_sundir);
        
        // diffuse model using directional derivatives.
        vec3 shade = g_suncol * g_sand_diff * max(f2 - f1, 0.0)/EPS;

        // lose energy based on density f1
        col = mix(col, shade, f1*smoothstep(0.1, -0.2, rd.y));
        
        // hashing to reduce banding
        t *= 0.9*(0.9 + 0.1*hash(t));
    }    
    return col;
}
{% endhighlight %}

{% include figure image_path="/assets/blog/sunfall/sunfall-12.jpg" caption="Volumetric dust without the terrain" %}