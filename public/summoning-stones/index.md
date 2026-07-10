# Summoning Stones


## Introduction
For my fourth and final block in my first year at [BUAS](https://www.buas.nl/) we had to work in teams and create a complete game in 8 weeks and put it on [Itch.](https://buas.itch.io/venture)\
This was my first time working on a game in a bigger team, this was a really fun experience and thought me a lot about teamwork and the other disciplines: design & art. I worked mostly on gameplay systems for the actual duration of the block, implementing the quest system, minion interactions, inventory system, initial UI setup and other things based on designer mockups. After the official 8 weeks of the project passed some students kept working on the project and thats when I started working on procedural level generation with designer [Thijs Werkman](https://www.linkedin.com/in/thijswerkman/). This is what the rest of the blog will be about😁
Make sure to play our game [**on Itch!!!!!!!**](https://buas.itch.io/venture)
## My final result
{{< youtube id=k4c949nH2sY  >}}
## Project features

- OpenGL compute path tracer
- Model loading
- Wavefront style path tracing
- Russian roulette
- Stochastic light sampling
- SVGF Denoising
- Sequential Impulse Solver
- Acceleration structures:
	- Bounding Volume Hierarchy
	- Brick mapping


## GPU ray tracing methods
##### Fragment
There are multiple ways to do ray tracing these days, you can either do it in a **fragment shader** like the original Teardown does, there are two blogs about how they do it from [Juan Diego Montoya](https://juandiegomontoya.github.io/teardown_breakdown.html) and [Acko](https://acko.net/blog/teardown-frame-teardown/).
##### Compute
However most people do it in a **compute shader** this is done by calling a compute shader invocation for each pixel on the screen then tracing a ray through the grid and then storing the final color in the pixel.
##### HWRT
There is also the option to use **Hardware ray tracing** with custom intersection shaders, You can do this if you have many different grids, which have a different orientation like the original Teardown does. This is also why Tuxedo Labs (the company owning Teardown) is using it for their new game.
You can find more information on what the difference is between their old and new renderer in [this talk](https://www.youtube.com/watch?v=IM1Dr98f3xU) given by Dennis Gustafsson and Gabe Rundlett at GPC 2025.
## Simple ray tracing
You can do a simple ray tracing algorithm where you get the albedo of the first voxel that you hit and then shoot another ray to a light in the scene. If the ray from the hit-voxel to the light does not hit anything, you multiply lightColor * voxel and store it in the pixel.\
You can then get an image like this.\
![raytracing](/images/voxel_teardown/compute_shader_2volumes.png)\
If you have any interest in how this all works visit [Jacco's blog](https://jacco.ompf2.com/2024/04/24/ray-tracing-with-voxels-in-c-series-part-1/) he has a really great series on how to trace voxels, he gives you a CPU template and some fun challenges😁. This is also how I got started initially.

## Voxel traversal algorithm
If you want to look into the algorithm that is used to traverse voxel grids. Look at this post by [Max](https://m4xc.dev/articles/amanatides-and-woo/) explaining how the [Amanatides and Woo's fast voxel traversal algorithm](https://www.researchgate.net/publication/2611491_A_Fast_Voxel_Traversal_Algorithm_for_Ray_Tracing) works. He shows some nice pictures which can give you a good intuition.

## Voxel Model Loading
The biggest file format for voxel worlds is by far .vox, these are models made in [MagicaVoxel](https://ephtracy.github.io/) there are multiple ways to load these models, you can try to write your own(don't do it) or use someone else's. I would heavily recommend using [ogt_vox.h](https://github.com/jpaver/opengametools/blob/master/src/ogt_vox.h) a single header parser which most of my classmates ended up using. There is also [gvox](https://github.com/GabeRundlett/gvox) “gvox is a meta-format that allows for several voxel data structures to co-exist within a single file.”
#### Here a castle model loaded:
![magica-voxel](/images/voxel_teardown/magica_voxel_model.png)
## Path tracing
I really wanted to do path tracing since I wanted to get Global Illumination, and it's the best way to simulate real life's lighting. By doing **Monte Carlo** path tracing we can simulate realistic environments and create photo-realistic images. We do this by simulating how light bounces around in the scene. If you want a more in-depth explanation about Path tracing look at [scratchapixel's article](https://www.scratchapixel.com/lessons/3d-basic-rendering/global-illumination-path-tracing/global-illumination-path-tracing-practical-implementation.html). I will now list a couple of ways to do Global Illumination:
- Voxel cone tracing
- Cellular automata
- Per voxel GI
- DDGI probes
- Radiance cascades
- ReSTIR GI
- Path tracing

I would like to go more into depth about some of these in a future article when I implement some.
#### This room doesn't have any direct lighting, the voxels only have emission and get randomly sampled similar to the skybox.
![lit-using-global-illumination](/images/voxel_teardown/lit_using_gi.png)
## Denoising using SVGF
SVGF stands for Spatiotemporal Variance Guided Filtering. Here **Spatiotemporal** means filtering across time and space. So we filter across neighbouring pixels and we look back into the past to check what colors we found at the same position in the previous frame. We can accurately get previous samples by standing still or reprojecting the view matrix of the current frame onto the previous frame (or use a frustum as mentioned in Jacco's blog). How many pixels we moved to the left/right or up/down will often be stored on a motion vector buffer. Getting pixels from previous frames and projecting them onto the new frame is called reprojection. A great blog by Jacco about it [here](https://jacco.ompf2.com/2024/01/18/reprojection-in-a-ray-tracer/)
An example of the denoised result using SVGF.

![reprojection](/images/voxel_teardown/reprojection.png)

Then we get to **Variance Guided Filtering**, which means we are filtering based on the difference of Variance between neighbouring and previous frame's pixels. If there is a high variance between those pixels we apply a heavier blur. Depending on the settings of your filtering you could either trust the past pixels a lot or little. For example you could take 0.8 * pastPixel and 0.2 * newPixel.
Or 0.99 * pastPixel and only 0.01 * newPixel. The latter would cause you to have laggy updates when light changes and might introduce motion blur/temporal lag. However if you trust the new pixels too much you will instead have a noisier output.
Here is an example of a raw path traced image and then a denoised one.

![pathtraced-raw](/images/voxel_teardown/pathtrace_raw.png)
![path-traced denoised](/images/voxel_teardown/pathtrace_denoised.png)

There are many more things that can be said about SVGF some links I found helpful are:
[TeamWisp.Github](https://teamwisp.github.io/research/svfg.html) which has links to the [SVGF paper](https://research.nvidia.com/sites/default/files/pubs/2017-07_Spatiotemporal-Variance-Guided-Filtering%3A//svgf_preprint.pdf) and [A-SVGF paper](https://cg.ivd.kit.edu/publications/2018/adaptive_temporal_filtering/adaptive_temporal_filtering.pdf). And I really enjoyed [jacquespillet's github explanation](https://github.com/jacquespillet/SVGF) on how he implemented SVGF.
And I might make a more in-depth article when I get started on NVIDIA's NRD denoising library. I'm particularly interested in trying the ReLAX denoiser. Later I'd like to look at neural denoising as well.
## Acceleration Structures
For acceleration structures I used a **BVH** (Bounding Volume Hierarchy) I first wrote my own one but when using the tinybvh library afterwards my bvh traversal speed went 2x so I swapped to that library :P (don't reinvent the wheel). Jacco Bikker has a great series on it again 😁 ["How to build a BVH 8 part series"](https://jacco.ompf2.com/2022/04/13/how-to-build-a-bvh-part-1-basics/), ["tinybvh manual"](https://jacco.ompf2.com/2025/01/24/tinybvh-manual-basic-use/) and ["BVH Quality: Beyond SBVH"](https://jacco.ompf2.com/2025/05/20/bvh-quality-beyond-sbvh/).
### Performance: NO BVH 41ms
![no_bvh](/images/voxel_teardown/no_bvh.png)
### With BVH 1ms
![bvh](/images/voxel_teardown/bvh.png)


I also used a Brickmap for much better empty space skipping of voxels. 
Here is an extreme example of the performance where I also happen to do multiple rays per pixel:
### Without Brickmap: 65ms
![without brick](/images/voxel_teardown/no_bricks.png)
### With Brickmap 16.7ms
![with brick](/images/voxel_teardown/bricks.png)

There are many different acceleration structures you could use all having different advantages such as storage size, general traversal speed and case specific traversal speed. Some of the examples are on this [bink website](https://bink.eu.org/fast-voxel-datastructures/) most notable being:
- BVH
- Brickmap
- Grid Hierarchy
- Sparse Voxel Octree
- Sparse 64 Tree
- VDB
- SDF

A newer one is NAADF (2026), which uses nested axis-aligned distance fields combined with other techniques.
## Further Optimizations
I got some further optimization by doing some heavy profiling and mostly looking at what other path tracers were doing to improve performance. I did some **Wavefront path tracing** which helped speed up the performance a bit, however this doesn't always work. I think it depends on your GPU's memory writing speed as it is very tough on the memory. Because when I turn OFF the performance mode of my laptop it is actually 0.2ms slower. But if I turn on max performance of my laptop I gain some performance:
### Without Wavefront 7.3ms
![perfmodestandard](/images/voxel_teardown/perf_mode_standard.png)
### With Wavefront 6.4ms
![perfmodewavefront](/images/voxel_teardown/perf_mode_wavefront.png)

If you want to learn more about Wavefront path tracing I would highly recommend this blog by [Jacco Bikker: Wavefront Path Tracing](https://jacco.ompf2.com/2019/07/18/wavefront-path-tracing/)😆.

And a final noticeable optimization was changing from storing integer values per voxel to bytes per voxel, which gained me 0.2ms. However that is from 1.5 to 1.3 making it **15% faster**. (if not more because of swap buffer and frame setup overhead). This is because my main bottleneck was texture/buffer reads on the GPU.
### Integer performance 1.5ms
![intperf](/images/voxel_teardown/intpervoxel.png)
### Byte performance 1.3ms
![byteperf](/images/voxel_teardown/bytepervoxel.png)

## Afterword
I would like to go into depth about denoising, GI and data structures in some later articles when I have done some more advanced things. There are also some other optimizations that I have done also regarding SVGF, but they are very specific to my project as it gives up some quality which is not really noticeable in my case, but the performance improvements are.

In the near future I would like to get started with Vulkan, doing some hardware raytracing on triangles and getting my hands dirty on ReSTIR. In the meantime I'm currently working on an Unreal Engine School project with a cool team of students. I will make a blog post about that later 😁. If you read this far thank you for reading!!! and if you have any questions shoot me a message somewhere on LinkedIn or Email (both are on the homepage).





