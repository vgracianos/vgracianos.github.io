---
title: "Segregation of Robotic Swarms"
excerpt: "A simple and effective controller that segregates a heterogeneous swarm into homogeneous clusters."
date: 2014-06-8
header:
  teaser: assets/research/segregation.jpg
---

{% include video id="tN6yEOUU00I" provider="youtube" %}

Abstract
--------

Several natural systems adopt self-sorting mechanisms based on
segregative behaviors. Among these, cell segregation is of particular
interest since it plays an important role in the formation of tissues,
organs, and living organisms. The Differential Adhesion Hypothesis
states that cells naturally segregate because of differences in
affinity, which lead similar cells to strongly adhere to each
other. By exploring this principle, we propose a controller that can
segregate a heterogeneous swarm of robots according to the
characteristics of each agent, such that similar robots form
homogeneous teams and dissimilar robots are segregated. We apply
LaSalle's Invariance Principle to show convergence and perform
simulated experiments in order to demonstrate the robustness and
effectiveness of the proposed controller. Results show that our
approach allows a swarm of multiple heterogeneous robots to segregate
in a coherent and smooth fashion, without any inter-agent collisions.

Publications
------------
* Santos, V.G.; Pimenta, L.C.A.; Chaimowicz, L., "Segregation of multiple heterogeneous units in a robotic swarm," IEEE International Conference on Robotics and Automation (ICRA) 2014, pp.1112,1117. [DOI](http://dx.doi.org/10.1109/ICRA.2014.6906993){: .btn .btn--primary} [PDF](https://dl.dropboxusercontent.com/s/rl3wiunxunqty7p/icra2014-segregation.pdf?dl=1){: .btn .btn--info}

Source Code
-----------
* [<i class="fab fa-fw fa-github" aria-hidden="true"></i> Github](https://github.com/vgracianos/segregation-icra){: .btn .btn--primary}
* [<i class="fa fa-fw fa-eye" aria-hidden="true"></i> Shadertoy](https://www.shadertoy.com/view/4s33Dj){: .btn .btn--primary}
