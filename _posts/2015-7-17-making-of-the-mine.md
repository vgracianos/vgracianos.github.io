---
title: "The Mine"
excerpt: "I breakdown one of my shaders and explain how the scene was modeled"
date: 2015-7-17
categories: blog
tags: [shadertoy]
header:
  teaser: assets/blog/themine/teaser.jpg
---

This is a brief summary of the modeling process behind [The Mine](https://www.shadertoy.com/view/XdfXzS) at Shadertoy.

<figure>
  <iframe width="100%" height="360" frameborder="0" src="https://www.shadertoy.com/embed/XdfXzS?gui=false&t=10&paused=false&muted=false" allowfullscreen></iframe>
</figure>

 The first objects that I added into the scene were the two rails. They are a pair of boxes that extend to infinity along the forward axis. I computed the distance to only the box on the right, and then I mirrored the space so as to achieve symmetry. I also carved an infinite cylinder with a very small radius at the inner part of the rails to add some detail.

{% include figure image_path="/assets/blog/themine/rails.jpg" caption="Infinite rails along the Z axis. Notice the carving at the inner sides." %}

The code of the distance function to the rails is straightforward:

{% highlight glsl  %}
// distance from p to box with width b.x, height b.y, and border radius r.
float box(vec2 p, vec2 b, float r) {
  return length(max(abs(p)-b, 0.0)) - r;
}

float rails(vec3 p) {
  // mirrors the x axis and translates.
  vec2 q = vec2(abs(p.x)-1.0, p.y + 1.7);
  float d = box(q, vec2(0.1), 0.01); q.x += 0.2;
  // carves the cylinder.
  return max(d, 0.11 - length(q));
}
{% endhighlight %}

Below the rails I included some planks. They are rectangular prisms that repeat along the forward axis. Repetition can be achieved by applying modular arithmetic on the distance function of a box.

{% highlight glsl  %}
float box(vec3 p, vec3 b, float r) {
  return length(max(abs(p)-b, 0.0)) - r;
}

float plank(vec3 p) {
  vec3 q = vec3(p.x, p.y + 1.9, mod(p.z, 2.0) - 1.0);
  return box(q, vec3(1.5, 0.05, 0.2), 0.01);
}
{% endhighlight %}

<figure class="half">
  <img src="/assets/blog/themine/planks-1.jpg">
  <img src="/assets/blog/themine/planks-2.jpg">
  <figcaption>Planks are repeated using modular arithmetic</figcaption>
</figure>

The supports of the mineshaft are boxes that span to infinity on the appropriate axis. I used a [shear mapping](http://en.wikipedia.org/wiki/Shear_mapping) place them along a diagonal direction. Repetition is achieved by the use of mirroring and modular arithmetic.

{% highlight glsl  %}
float support(vec3 p) {
  // translation, mirroring, and modulo.
  const vec4 c = vec4(0.15, 0.2, 2.0, 1.2);  
  vec3 q = vec3(abs(p.x) - c.z, p.y - c.z, mod(p.z, 6.0) - 3.0);
  float d = box(q.xz, c.xx, 0.05);   // vertical support.
  d = min(d, box(q.yz, c.yx, 0.05)); // horizontal support.
  q.x += q.y + c.w; // shear mapping
  return min(d, box(q.xz, c.xx, 0.05)); // diagonal support.
}
{% endhighlight %}

<figure class="third">
  <img src="/assets/blog/themine/support-1.jpg">
  <img src="/assets/blog/themine/support-2.jpg">
  <img src="/assets/blog/themine/support-3.jpg">
  <figcaption>Three infinite boxes are used to model the support structure</figcaption>
</figure>

<figure class="half">
  <img src="/assets/blog/themine/support-4.jpg">
  <img src="/assets/blog/themine/support-5.jpg">
  <figcaption>Mirroring and modular arithmetic create repetition</figcaption>
</figure>

The last part was modelling the cave itself. I placed an infinite cylinder around the track as such that it cuts through the supports to hide their prolongations. I used a [superquadric](http://en.wikipedia.org/wiki/Superquadrics) to have control over the roundness of the cave, and [fractional brownian motion](https://code.google.com/p/fractalterraingeneration/wiki/Fractional_Brownian_Motion) noise was used to add details to the walls.

{% highlight glsl  %}
float cave(vec3 p) {
  // superquadric
  const float k = 4.0;
  return 1.6-pow(pow(abs(p.x), k) + pow(abs(p.y), k), 1.0/k);
}

vec2 dist_field(vec3 p) {
  // warps the XY plane as a function of Z.
  p.xy += track(p.z); 

  // displacement mapping based on fractal brownian motion.
  float dist = cave(p) + fbm(p);
 
  dist = min(dist, support(p));
  dist = min(dist, plank(p));
  dist = min(dist, rails(p)); 
  return dist;
}
{% endhighlight %}

<figure class="half">
  <img src="/assets/blog/themine/cave-1.jpg">
  <img src="/assets/blog/themine/cave-2.jpg">
  <figcaption>Superquadric surrounding the rails and support. Fractional brownian motion adds lots of detail inside the model.</figcaption>
</figure>

The `track` function in the above code distorts the space as a function of depth to create a sense of an uneven path. I used a simple sum of sines and cosines to achieve the distortion.

{% highlight glsl %}
vec2 track(float z) {
  float x = cos(0.2*z);
  float y = -cos(0.2*z) - 0.1*sin(0.8*z - 2.0);
  return vec2(x, y);
}
{% endhighlight %}

{% include figure image_path="/assets/blog/themine/track.jpg" caption="Sines and cosines distort the space along the forward axis" %}

And that's all of the modeling! As you can see, I've used only simple shapes together with [Constructive Solid Geometry](http://en.wikipedia.org/wiki/Constructive_solid_geometry) techniques to build the whole scene. I think it is really interesting how just a few lines of codes can add a lot to the output image. The shading of these models is based on a simple Lambertian and Phong model, and I used cube mapping to apply textures onto them. There's nothing fancy, so I think most people can have a look at [the code](https://www.shadertoy.com/view/XdfXzS).
