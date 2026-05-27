# Real-time Voxel Path tracer


## Introduction
For my third block in my first year at [BUAS](https://www.buas.nl/) the objective was to create a ray-tracer in **6 Weeks**, and spend the remaining **2 weeks** building a game. \
We got a template and were going to do CPU ray-tracing this block, however since I knew from the beginning that I would need the CPU for the heavy physics calculations I wanted to do the rendering on the GPU. This is because I wanted to create a teardown like voxel-game. So I asked my lecturer if I could work on the **GPU** and he said yes🥳! \
If you want to see the other projects made here is a [LinkedIn post](https://www.linkedin.com/feed/update/urn:li:activity:7449112698362978304/?utm_source=share&utm_medium=member_desktop&rcm=ACoAADr6NbMBO3mTgVuLj_NaTHtbo8UwplCHnXw) from my lecturer Jacco Bikker, below the video I will explain my journey through the project and things I learned.\
[Code is on Github](https://github.com/TimonVenninckx/Teardown-voxel-path-tracer) it's not pretty but I will be focussing on writing cleaner code/API's for future projects.
## My final result
{{< youtube id=k4c949nH2sY  >}}
## Project features

- OpenGL compute path tracer
- Model loading
- Wavefront style path-tracing
- Russian roulette
- Stochastic light sampling
- SVGF Denoising
- Sequential Impulse Solver
- Acceleration structures:
- Bounding Volume Hierarchy
- Brick mapping


## GPU ray tracing methods
##### Fragment
There are multiple ways to do ray-tracing these days, you can either do it in a **fragment shader** like the original Teardown does, there are two blogs about how they do it from [Juan Diego Montoya](https://juandiegomontoya.github.io/teardown_breakdown.html) and [Acko](https://acko.net/blog/teardown-frame-teardown/).
##### Compute
However most people do it in a **compute shader** this is done by calling a compute shader invocation for each pixel on the screen then tracing a ray through the grid and then storing the final color in the pixel.
##### HWRT
There is also the option to use **Hardware ray tracing** with custom intersection shaders, You can do this if you have many different grids which have a different orientation like the original teardown does, this is also why Tuxedo-Labs(the company owning teardown) is using it for their new game.
You can find more information on what the difference is between their old and new renderer in [this talk](https://www.youtube.com/watch?v=IM1Dr98f3xU) given by Dennis Gustafsson and Gabe Rundlett at GPC2025.
## Simple ray tracing
You can do a simple ray-tracing algorithm where you get the albedo of the first voxel that you hit and then shoot another ray to a light in the scene. If the ray from the hit-voxel to the light does not hit anything, you multiple lightColor * voxel and store it in the pixel.\
You can then get an image like this.\
![raytracing](/images/voxel_teardown/compute_shader_2volumes.png)\
If you have any interests in how this all works visit [Jacco's blog](https://jacco.ompf2.com/2024/04/24/ray-tracing-with-voxels-in-c-series-part-1/) he has a really great series on how to trace voxels, he gives you a CPU template and fun challenges to do😁. This is also how I got started initially.

## Voxel traversal algorithm
If you want to look into the algorithm that is used to traverse voxel grids. Look at this post by [Max](https://m4xc.dev/articles/amanatides-and-woo/)explaining how the [Amanatides and Woo's fast voxel traversal algorithm](https://www.researchgate.net/publication/2611491_A_Fast_Voxel_Traversal_Algorithm_for_Ray_Tracing) works. He shows some nice pictures which can give you a good intuition.

## Voxel Model Loading
The biggest file format for voxels worlds is by far .vox, these are models made in [MagicaVoxel](https://ephtracy.github.io/) there are multiple ways to load these models, you can try to write your own(don't do it) or use someone else's. I would heavily recommend using [ogt_vox.h](https://github.com/jpaver/opengametools/blob/master/src/ogt_vox.h) a single header parser which most of my classmate's ended up using. There is also [gvox](https://github.com/GabeRundlett/gvox)“gvox is a meta-format that allows for several voxel data structures to co-exist within a single file.”
#### Here a castle model loaded:
![magica-voxel](/images/voxel_teardown/magica_voxel_model.png)
## Path tracing
I really wanted to do path tracing since I wanted to get Global Illumination, and it's the best way to simulate real-life's lighting. By doing Monte Carlo path tracing we can simulate realistic environments and create photo-realistic images. We do this by simulating how light bounces around in the scene. If you want a more in-depth explanation about Path tracing look at [scratchapixel's article](https://www.scratchapixel.com/lessons/3d-basic-rendering/global-illumination-path-tracing/global-illumination-path-tracing-practical-implementation.html). I will now list a couple of ways to do Global Illumination:
- Voxel cone tracing
- Cellular automata
- Per voxel GI
- DDGI probes
- Radiance cascades
- Restir GI
- Path-Tracing

I would like to go more into depth about some of these in a future article when I implement some.
#### This room doesn't have any direct lighting, the voxels just have emission and can get randomly sampled similar to the skybox.
![lit-using-global-illumination](/images/voxel_teardown/lit_using_gi.png)
## Denoising using SVGF
SVGF stands for Spatiotemporal Variance Guided Filtering. Here **Spatiotemporal** means filtering across time and space. So we filter across neighbouring pixels and we look back into the past to check what colors we found at the same position in the previous frame. We can accurately get previous samples by standing still or reprojecting the view matrix of the current frame onto the previous frame (or use a frustum as mentioned in Jacco's blog below). How many pixels we moved to the left/right or up/down will often be stored on a motion vector buffer. Getting pixels from previous frames and projecting them onto the new frame is called reprojection. A great blog by Jacco about it [here](https://jacco.ompf2.com/2024/01/18/reprojection-in-a-ray-tracer/)
An example of the denoised result using SVGF.
![[reprojection.png]]

Then we get to Variance Guided Filtering, which means we are filtering based on the difference of Variance between neighbouring and previous frame's pixels. If there is a high variance between those pixels we apply a heavier blur. Depending on the settings of your filtering you could either trust the past pixels a lot or little. For example you could take 0.8 * pastPixel and 0.2 newPixel.
or 0.99 * pastPixel and only 0.01 * newPixel. The later would cause you to have laggy updates when light changes and might introduce motion blur/temporal lag. However if you trust the new pixels too much you will instead have a noisier output.
Here is an example of a raw path traced image and then a denoised one.

![pathtraced-raw](/images/voxel_teardown/pathtrace_raw.png)
![path-traced denoised](/images/voxel_teardown/pathtrace_denoised.png)

There are many more things that can be said about SVGF some links I found helpful are:
[TeamWisp.Github](https://teamwisp.github.io/research/svfg.html) which has links to the [SVGF paper](https://research.nvidia.com/sites/default/files/pubs/2017-07_Spatiotemporal-Variance-Guided-Filtering%3A//svgf_preprint.pdf) and [A-SVGF paper](https://cg.ivd.kit.edu/publications/2018/adaptive_temporal_filtering/adaptive_temporal_filtering.pdf). And I really enjoyed [jacquespillet's github explanation](https://github.com/jacquespillet/SVGF) on how he implemented SVGF.
And I might make a more in-depth article when I get started on NVIDIA's NRD denoising library. I'm particularly interested in trying the RELAX denoiser. And maybe later visiting neural denoising.
## Acceleration Structures
For acceleration structures I used a **BVH** (Bounding Volume Hierarchy) I first wrote my own one but when using the tinybvh library afterwards my bvh traversal speed went 2X so I swapped to that library :P (don't reinvent the wheel). Jacco Bikker has a greet series on it again 😁 ["How to build a BVH 8 part series"](https://jacco.ompf2.com/2022/04/13/how-to-build-a-bvh-part-1-basics/) , ["tinybvh manual"](https://jacco.ompf2.com/2025/01/24/tinybvh-manual-basic-use/) and ["BVH Quality: Beyond SBVH"](https://jacco.ompf2.com/2025/05/20/bvh-quality-beyond-sbvh/).
Performance increase without BVH and with can be seen below.


I also used a Brickmap for much better empty space skipping of voxels.


## Further Optimizations



## Afterword






