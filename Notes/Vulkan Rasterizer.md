# What is the Rasterizer
>The rasterizer is a [[Fixed Functions|Fixed Function]] that converts the 3D geometry of a scene into pixel fragments that are then passed off to the fragment shader. 

Its created via a `vk::PipelineRasterizationStateCreateInfo` and passed to the [[Vulkan Pipeline|Graphics Pipeline]]. The default `vk::PipelineRasterizationStateCreateInfo` looks like this:
```cpp
vk::PipelineRasterizationStateCreateInfo rasterizer{
	.depthClampEnable        = vk::False,
	.rasterizerDiscardEnable = vk::False,
    .polygonMode             = vk::PolygonMode::eFill,
    .cullMode                = vk::CullModeFlagBits::eBack,
    .frontFace               = vk::FrontFace::eClockwise,
    .depthBiasEnable         = vk::False,
    .lineWidth               = 1.0f
};
```
---
## Depth Clamp
>The depth clamp says that if geometry falls outside the view of the player on the Z axis instead of it being culled it will be brought to the nearest  point inside the near and far clipping plane. 

Why would we do this? For more advanced shadow techniques this can be used to reduce pop-in of shadows if an object leaves the near and far plane of the player. 

## Rasterizer Discard
This is effectively an "off switch" for the rasterizer, making it so that nothing will be outputted to the frame buffer.
## Polygon Mode
This determines how polygons are converted into fragments with three options, `eFill` and `eLine` are obvious in use cases but `ePoint` isn't as clear.

| PolygonMode               | Description                                             |
| :------------------------ | ------------------------------------------------------- |
| `vk::PolygonMode::eFill`  | Fll the area of the polygon with fragments **(normal)** |
| `vk::PolygonMode::eLine`  | Polygon edges are drawn as lines **(wireframe)**        |
| `vk::PolygonMode::ePoint` | Polygon vertices are drawn as points.                   |
It should be noted that both `eLine` and `ePoint` requires enabling a GPU feature.

## Cull Mode
This determines what faces to cull. Default is `vk::CullModeFlagBits::eBack`, however, you could cull the front, back, both or neither. It is a bitmask so culling both would be `(vk::CullModeFlagBits::eBack | vk::CullModeFlagBits::eFront)`

## Front Face
> This specifies the vertex order for a face to be considered front-facing (Clockwise vs. Counter-Clockwise).

Vulkan uses the **Winding Order** of vertices in 2D screen space to determine which side of a triangle you are looking at. 

When we set `.frontFace = vk::FrontFace::eClockwise`, we are telling the hardware:

*"If the GPU processes Vertex 0 -> 1 -> 2 and they trace a **clockwise loop** from the screen's perspective, treat this as the front side. If they trace a counter-clockwise loop, it's the back side."*
#### Why do we do this??*
Every set of 3 points actually defines one triangle with two side. One side that has its normal vector pointing forward, and one side  that has its normal vector pointing backwards.
 
 We only want to render one of these which is what our Index buffer helps us do, helping us pick a "front," *but wait,* **there's a problem!**

The index buffer only tells us which triangles are on the "outside" of our mesh, but they don't tell us if we should be culling them right now! That's where winding order comes into play. 

If we traverse the triangle defined by the index buffer and it goes counter clockwise then the triangle is currently rotated and should be culled. 

## Line Width
How wide do we want our lines to be? Normally capped at 1.0f unless enabled with the `wideLines` GPU feature.