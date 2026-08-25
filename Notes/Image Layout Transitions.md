## What's an image layout? 
Texture and [[Swap Chain]] images aren't just massive 2d arrays of pixels, rather, they get stored in various formats. This is because not every image needs the same operations so we can get more performance if we specify.
## Transition?
> These layouts are not like traditional variables that we can use `static_cast<T>()` on. Rather they represent how certain images are stored on the [[GPU]] therefore we need to tell the GPU "hey I'm done using it like this please change it to be laid out in this new way."

## Stages
Every transition has a src stage and a dst stage. They work like the following.
- `src_stage_mask` *"What GPU work must finish before the barrier can execute?"*
- `dst_access_mask` *"Which future GPU stages must wait for this barrier to finish before they are allowed to start?"*
# Access
> This was extremely hard to understand when researching so I apologize if I butcher it.

Let's start with `oldLayout` and `newLayout`. The `oldLayout` is the previous layout our data was organized in, its optimized to do certain things. When we use the  `srcStageMask` we are telling that cache to flush its contents into the main [[VRAM]] where the image is stored. Note that it will match the `oldLayout`. The barrier then rearranges everything to match the `newLayout`, specifically rearranging everything you just flushed. We now invalidate the caches at `dstAccessMask` telling them to look to the main [[VRAM]] to load their caches. 

`dstAccessMask` is usually not the same as `srcAccessMask` as `dstAccessMask` has to match the `newLayout` and `srcAccessMask` has to match the `oldLayout`. For example, when you're reading a depth texture you're not accessing the same cache that the depth texture was originally written to as that cache is optimized for writing not reading.

One of these transitions looks like this:
```cpp
// SRC (Before the barrier): What did the GPU just do
// If we're initialzing then no operations are happening prior, therefore we don't // need to wait for anything to finish.
vk::PipelineStageFlags2 src_stage_mask  = vk::PipelineStageFlagBits2::eNone;
vk::AccessFlags2        src_access_mask = vk::AccessFlagBits2::eNone;

// DST (After the barrier): What is the GPU about to do
// Since we're writing colors to the image we'll want any ColorAttachment 
// operations to finish before the GPU can run any more commands.
vk::PipelineStageFlags2 dst_stage_mask  = 
	vk::PipelineStageFlagBits2::eColorAttachmentOutput;
	
vk::AccessFlags2        dst_access_mask = 
	vk::AccessFlagBits2::eColorAttachmentWrite;


vk::ImageMemoryBarrier2 barrier = {
    .srcStageMask        = src_stage_mask,          
    .srcAccessMask       = src_access_mask,        
    .dstStageMask        = dst_stage_mask,          
    .dstAccessMask       = dst_access_mask,     
    
    // We explicitly tell Vulkan: discard whatever junk data was there, 
    // and optimize the memory layout for drawing color pixels.
    .oldLayout           = vk::ImageLayout::eUndefined,
    .newLayout           = vk::ImageLayout::eColorAttachmentOptimal,
    
    // We aren't transferring ownership between different queue families 
    .srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED,
    .dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED,
    
    // The handle to the specific swapchain image we are currently working on
    .image               = swapChainImages[imageIndex],
    
    // Target only the main color layer (no mipmaps, no 3D texture layers)
    .subresourceRange    = {
        .aspectMask     = vk::ImageAspectFlagBits::eColor,
        .baseMipLevel   = 0,
        .levelCount     = 1,
        .baseArrayLayer = 0,
        .layerCount     = 1
    }
};

vk::DependencyInfo dependency_info = {
	// No special flags (like regional dependency) needed here
    .dependencyFlags         = {},
    .imageMemoryBarrierCount = 1,
    .pImageMemoryBarriers    = &barrier
};

//add this command into the command buffer. It's called a barrier as it will prevent the execution of subsequent operations until its complete.
commandBuffer.pipelineBarrier2(dependency_info);
```

