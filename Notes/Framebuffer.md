# What is the Framebuffer
> The framebuffer is the part of your [[GPU]]'s memory ([[VRAM]]) that holds the next image your computer should display. The [[GPU]] constantly reads from this data to display images on your screen.

# What's inside the Framebuffer?
The frame buffer isn't really *one* singular image, its rather a collage of different buffers that we call "attachments" that are bound together.
## The Color Buffer
This is most likely what you immediately think of, a giant 2D array that holds all the [[RGBA]] colors you see on screen. 

## The Depth Buffer
The [[Depth Buffer]] is a greyscale map that tells us how far away any pixel is from the camera. This is helpful for resolving ordering of multiple objects in a scene. As the pixels grow darker (i.e. closer to the zero vector) they grow closer to the camera.
## The Stencil Buffer
The [[Stencil Buffer]] is a masking layer we use to discard certain pixels. This is commonly used for mirrors, shadows, and outlines. 

# What is Vulkan's Framebuffer?
In Vulkan a `vk::Framebuffer` is only a immutable link between a [[Render Pass]] to a [[Image View]]. It's effectively a contract saying "when this render pass renders something it will print to this image view."

We initialize it with a `vk::FramebufferCreateInfo` struct. 
```c++
vk::FramebufferCreateInfo framebufferInfo{
    .renderPass = renderPass,          // The blueprint
    .attachmentCount = 1,              
    .pAttachments = &imageView,        // The actual memory allocation
    .width = swapChainExtent.width,
    .height = swapChainExtent.height,
    .layers = 1
};
```

# Swap Chaining 
We use swap chaining in Vulkan so that we can do double buffering. We work on one frame while the previous frame is being shown. This prevents screen tearing as we only *swap* to the new frame when its done.