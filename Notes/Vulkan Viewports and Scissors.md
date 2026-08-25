# What is the Viewport
>When our [[Vertex Shader]] processes geometry it spits out [[Normalized Device Coordinates]] (NDC). These do not match the coordinate plane of our screens, therefore we need a scaling operation to turn our [[Normalized Device Coordinates|NDC]] coordinates to locations in the [[Framebuffer]]

We can create a viewport with the following struct.

```cpp
vk::Viewport viewport{
    .x = 0.0f, //left most point
    .y = 0.0f, //top most point
    .width = static_cast<float>(swapChainExtent.width),
    .height = static_cast<float>(swapChainExtent.height),
    .minDepth = 0.0f, //z value
    .maxDepth = 1.0f
};
```

The width and height of a Viewport will almost be the width and height of the window and start at (0,0). The viewport fundamentally is a linear transformation specifically a scale.  
# What are scissors?
> If the viewport scales the input to make it fit a certain view, scissors **cut** anything outside the bounds of a certain view. It is a **destructive** operation.

# Dynamic bindings
It's very common for us to change our viewports or scissors on the fly. (For example take resizing a window). Therefore, it's very common to have them as a dynamic state set in the [[Command Buffers|command buffer]]. This is possible without performance loss.

The standard way to do this is seen below
```cpp
std::vector<vk::DynamicState> dynamicStates = {vk::DynamicState::eViewport, vk::DynamicState::eScissor};

vk::PipelineDynamicStateCreateInfo dynamicState{.dynamicStateCount = static_cast<uint32_t>(dynamicStates.size()), .pDynamicStates = dynamicStates.data()};
```

If we do this, then we only need to list the viewport and scissors count when making our `vk::PipelineViewportStateCreateInfo` object.

```cpp
vk::PipelineViewportStateCreateInfo viewportState{.viewportCount = 1, .scissorCount = 1};
```

If they are statically generated, then both rectangles need to be set in the pipeline when we create it.

```cpp
vk::PipelineViewportStateCreateInfo viewportState{.viewportCount = 1, .pViewports = &viewport, .scissorCount = 1, .pScissors = &scissor};
```