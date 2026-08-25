# What is a fixed function??
> A fixed function is a **hardware circuit** on your [[GPU]] that has a fixed "function" and are not programmable like the [[Vertex Shader]] or the [[Fragment Shader]]. 
## What is an FF in Vulkan
In Vulkan (like everything) we precompute our shaders for speed. This means that if we change literally any of the definitions of these FF stages that the entire [[Vulkan Pipeline|pipeline]] needs to be remade. 
## What if I know something is gonna be dynamic?
Luckily Vulkan lets us exclude certain states if we want at the cost of speed. A common example of this would be if we want to remove the viewport and scissor circuits to allow the window to be freely resized.
>[!example] Dynamic State Struct
>The following code makes a vector which says "exclude the viewport and scissors fixed function circuits from the graphics pipeline so they can be dynamic."
>```c++
>std::vector<vk::DynamicState> dynamicStates = {
>	vk::DynamicState::eViewport,
>	vk::DynamicState::eScissor
>};
>vk::PipelineDynamicStateCreateInfo dynamicState {
>	.dynamicStateCount = static_cast<uint32_t>(dynamicStates.size()),
>	.pDynamicStates = dynamicStates.data()
>};
>```

# Shader Type
> In Vulkan, fixed-function stages are non-programmable hardware circuits baked into the `VkPipeline`. If we change any non-dynamic state, the entire pipeline must be recreated.

---

| Stage                                                     | Summary                                                                                                                                                                                                                                                                             |
| :-------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [[Vulkan Vertex Input\|Vertex Input]]                     | Defines the layout of raw 3D model data (coordinates, colors) fed to the [[Vertex Shader]].                                                                                                                                                                                         |
| [[Vulkan Input Assembly\|Input Assembly]]                 | Defines how the renderer should handle the vertices you give it. E.g. "Is this a list of points, lines, or triangles?"                                                                                                                                                              |
| [[Vulkan Viewports and Scissors\|Viewports and Scissors]] | **Viewport** is the region of the [[Framebuffer]] that the output will be rendered to. This is like the canvas your graphics card is painting to. **Scissors** **cut** anything outside of their area. View port is like a scaling operation, and scissors is like a cut operation. |
| [[Vulkan Rasterizer\|Rasterizer]]                         | Responsible for turning the 3D geometry and turning it into 2D pixel fragments.                                                                                                                                                                                                     |
| [[Multisampling]]                                         | One of the ways to preform [[Antialiasing]], will be revisited and filled in later.                                                                                                                                                                                                 |
| [[Depth and stencil testing]]                             | Depth testing is used to resolve occlusion and stencil testing is used to cut out certain sections of the screen. The buffers for both can be found in the [[Framebuffer]] notes.                                                                                                   |
| [[Vulkan Color Blending\|Color blending]]                 | This handles combining the color outputted by the [[Fragment Shader]] and the color already existing in the [[Framebuffer]].                                                                                                                                                        |
| [[Pipeline layout]]                                       | Used for defining global `uniform` values in shaders which allow shaders to be altered at runtime without having to recreate them.                                                                                                                                                  |


