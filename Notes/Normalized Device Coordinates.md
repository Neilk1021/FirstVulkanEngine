# What is this?
Normalized device coordinates (NDC) are a standardized 3D coordinate system used by the GPU after the vertex shader. 

In Vulkan:
* **X-Axis:** -1.0 (Left) to 1.0 (Right)
* **Y-Axis:** -1.0 (Top) to 1.0 (Bottom) 
* **Z-Axis:** 0.0 (Near/Camera) to 1.0 (Far)

# Why do we have this?
Monitors come in thousands of different resolutions and aspect ratios. NDCs give the graphics pipeline a "universal canvas" to do all geometric math. Once the geometry is perfectly positioned inside this universal -1.0 to 1.0 box, the [[Vulkan Viewports and Scissors|View port]] transformation step steps in to map those coordinates to actual screen pixels (like 1920x1080).