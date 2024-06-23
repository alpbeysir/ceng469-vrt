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
of intersection checks. The GPU ray tracing pipeline has a similar concept named acceleration structures. Before rendering the scene, these acceleration structures are generated using the mesh data. Then when an intersection query
result is needed during rendering, the shaders access this structure to get the required data.

The generation of acceleration structures is done in two steps:

### Bottom-Level AS Generation

1. For each model, get the vertex and index buffers.
2. Allocate scratch memory on the GPU to be used for generation.
3. Fill a command buffer with AC generation and compaction commands.
4. Send the command buffer for execution on the GPU.

### Top-Level AS Generation

1. For each model instance, fill out a VkAccelerationStructureInstanceKHR struct.
2. Fill command buffer with necessary commands.
3. Upload the filled structs to the GPU.
4. Mark command buffer for execution.

The bottom level AS keeps vertex and intersection data and the top level AS keeps material & transform data for each model instance.

<div style="display: flex;">
  <img src="acc.png" alt="Image 2" style="flex: 85%; padding: 10px;">
</div>

## Descriptor Sets

Descriptor sets are basically the Vulkan equivalent of OpenGL SSBOs. In the ray tracing shaders, we need to access the previously generated acceleration structures and the off-screen buffer to which we will be rendering. In order to do that, we need to create a descriptor set with that information. 

// TODO explain in detail

## Ray Tracing Pipeline

Traditional graphics pipelines are sequential. A list of stages is given to the GPU and a shader is executed in each of these stages. The final output image is then displayed on the screen. Ray tracing pipelines do not have this property as any shader
in the pipeline may be needed at any time. This is because of the nature of ray intersection. When a ray is cast from a location, it is not known in advance which object -or the skybox- it will hit. Any object's shader output may be needed to process that ray's bounce. To model this process, a separate stage is added to the pipeline for each 'state' of a ray. These stages are as follows:

1. Ray generation: The initial ray casting from the camera happens here
2. Ray miss: Called by the GPU when a ray doesn't hit anything
3. Ray hit: Called by the GPU when an intersection test succeeds

Additional optional stages may be used depending on the complexity of the program.

### Shader Binding Table

When a ray is traced on the GPU, the information on which shader to call of which object is provided by the shader binding table (SBT). This SBT is created on the CPU and is basically an array of function pointers with specific alignment rules.
The GPU uses the SBT as a lookup table to know which shader function to invoke for a given ray hit (or miss). There should be a region of the SBT for each stage of the RT pipeline.

<div style="display: flex;">
  <img src="sbt.png" alt="Image 2" style="flex: 85%; padding: 10px;">
</div>

