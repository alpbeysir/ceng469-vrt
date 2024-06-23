# Computer Graphics II - Vulkan Ray Tracing

Traditional ray tracing methods are not hardware-accelerated. The ray generation and intersect tests are done manually. This leads to slowdowns when the scene contains many models or has a high primitive count.
Breakthroughs in modern graphics cards have led to the implementation of hardware ray tracing support. In this project, we demonstrate the advantages of using hardware ray tracing. The results show that performance is substantially higher compared
to previous approaches and that the implementation of ray tracing is simplified.

## Overview

Before explaining the specifics of hardware ray tracing, let's remember how we implemented CPU ray tracing in 477 HW1:

1. Loop through the pixels of the screen, emit one ray for each.
2. Cast each ray through the scene, check for intersections.
3. Handle reflections by casting rays from intersection points.
4. After intersection points are found, apply material equations and scene lights to get color output.

In hardware ray tracing, the above steps stay the same but the GPU 'helps' us with some of them.

## Acceleration Structures

In the above list, step 2 requires us to iterate through all the objects in the scene and run an intersection calculation for each. This becomes prohibitively expensive as the triangle count grows. On the CPU, BVH algorithm may be used to reduce the number
of intersection checks. The GPU ray tracing pipeline has a similar concept named acceleration structures. Before rendering the scene, these acceleration structures are generated using the mesh data and uploaded to the GPU. Then when an intersection query
result is needed during rendering, the shaders access this structure to get the required data.

<div style="display: flex;">
  <img src="acc.png" alt="Image 2" style="flex: 85%; padding: 10px;">
</div>

