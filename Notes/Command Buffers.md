# What is a command buffer?
> In [[Vulkan]] we don't directly do things like draw or move memory with function calls. We have to write down what operations we want to do in command buffer objects. This is so [[Vulkan]] can process our commands in batches which leads to some efficiency gains. Additionally this allows for [[Aysnc (CS)|asynchronous]] command recording.

# Command Pools
[[Vulkan Command Pools]] are responsible for handling the memory allocation of [[Command Buffers]], this is done for efficiency and is often hidden in other APIs.   
## How to make in Vulkan
```cpp
vk::CommandPoolCreateInfo poolInfo {
	.flags            = vk::CommandPoolCreateFlagBits::eResetCommandBuffer,
	.queueFamilyIndex = queueIndex
};

commandPool = vk::raii::CommandPool(device, poolInfo);
```

* `flags` is a [[Bitwise]] field that enables certain functionality in the pool. For detailed explanation checkout [[Vulkan Command Pool Flags|Flags]]. 
* `queueFamilyIndex` specifies what graphics family these commands fall under which is important as it determines what operations you can perform. 
# Command buffer allocation
To allocate a Command Buffer we first need to writ a `vk::CommandBufferAllocateInfo` to describe certain settings of our command buffer.
```cpp
vk::CommandBufferAllocateInfo allocInfo{ 
	.commandPool = commandPool, 
	.level = vk::CommandBufferLevel::ePrimary, 
	.commandBufferCount = 1 
};
//this bullshit is because vk::raii::CommandBuffers allocates comandBufferCount
//buffers and returns an vector of Command Buffers even if we only want one.
commandBuffer = std::move(vk::raii::CommandBuffers(device, allocInfo).front());
```

- `commandPool` is the [[Vulkan Command Pools|Command Pool]] we allocated in [[#Command Pools]]. 
- `level` basically describes "is this Command Buffer a library or executable?" This separation is so the hierarchy of dependencies is clean and acyclic.
	- `vk::CommandBufferLevel::ePrimary` means that we can execute this buffer but it cannot be called from other Command Buffers
	- `vk::CommandBufferLevel::eSecondary` means that we cannot execute this buffer but it can be called from other Primary Command Buffers. 
- `commandBufferCount` tells the allocator how many buffers of this type do we want.

# Command buffer recording
We start with this function where `imageIndex` is the image in the [[Swap Chain]] we want to write to.

```cpp
void recordCommandBuffer(uint32_t imageIndex)
{
}
```

To start recording to our command buffer we write `vk::raii::CommandBuffer::begin()` and since this is Vulkan we have to pass in a `vk::CommandBufferBeginInfo` struct as a parameter.
## Buffer Begin Info
> Buffer Begin Info contains two data members, `flags` and `pInheritanceInfo

- `flags` specifies how exactly we plan to use the command buffer.
	- `vk::CommandBufferUsageFlagBits::eOneTimeSubmit` says "hey this buffer is only going to be executed once and will be recorded after. "
	- `vk::CommandBufferUsageFlagBits::eRenderPassContinue` is for **Secondary** buffers and says "hey I need to be executed in the duration of a render pass." The reason for this is that **Secondary buffers do not control their own execution and cannot call `vkCmdBeginRenderPass` or  `vkCmdEndRenderPass`** If you do any draw calls in this buffer (like `vkCmdDraw` then this flag needs to be enabled).
	- `vk::CommandBufferUsageFlagBits::eSimultaneousUse` basically allows you to reuse the same [[Command Buffers]] in multiple places. However, this forces it to be read only. [[Command Buffers]] by default are kind of similar to locks/mutexes in that only one thing should have access to it at a time. 
- `pInheritanceInfo` is a field for Secondary Command buffers and is a pointer to a [[Vulkan Command Buffer Inheritance Info]] struct. At a high level it's acting as parameters to provide a Secondary Buffer context that the primary buffer has by default. 

If we already recorded this command buffer then `vk::raii::CommandBuffer::begin` will reset this command buffer as the commands in a buffer are immutable after a buffer is recorded.
# Image layout transitions
[[Image Layout Transitions]] are effectively maps that tell the GPU how to move from representing a particular image one way to another. We store images in different layouts to optimize for different operations.

For example we may move from `vk::ImageLayout::eUndefined` to `vk::ImageLayout::eColorAttachmentOptimal` which would mean "discard the old image data and layout the new image data in a way that makes it easy to change the color of pixels."
# Dynamic Rendering
We first have to swap the image layout of the current image in the [[Swap Chain]] to allow us to write colors to it.

```cpp
transitionImageLayout(  
    imageIndex,  
    vk::ImageLayout::eUndefined,  
    vk::ImageLayout::eColorAttachmentOptimal,  
    {},                                                        // srcAccessMask 
    vk::AccessFlagBits2::eColorAttachmentWrite,                // dstAccessMask  
    vk::PipelineStageFlagBits2::eColorAttachmentOutput,        // srcStage  
    vk::PipelineStageFlagBits2::eColorAttachmentOutput         // dstStage  
);  
```

We then have to specify what image we want to modify and prepare it. Here we paint the entire image black via the `loadOp` `vk::attachmentLoadOp::eClear`. Which fills the entire image with the `clearValue`. `storeOp` is used to specify if we want to get rid of or keep the image in [[VRAM]] after we render it.

```cpp
vk::ClearValue clearColor = vk::ClearColorValue(0.0f, 0.0f, 0.0f, 1.0f);  

vk::RenderingAttachmentInfo attachmentInfo = {  
    .imageView   = swapChainImageViews[imageIndex],    
    .imageLayout = vk::ImageLayout::eColorAttachmentOptimal,    
     //wipe out the previous image clear it    
    .loadOp      = vk::AttachmentLoadOp::eClear,    
     //please store this and don't wipe it immediately    
    .storeOp     = vk::AttachmentStoreOp::eStore,    
     //clear it with this color    
    .clearValue  = clearColor
};
```

We then have to specify the `RenderingInfo` for our buffer. 

```cpp
vk::RenderingInfo renderingInfo = {
    .renderArea           = 
    {
	    .offset = {0, 0}, 
	    .extent = swapChainExtent
	},
    .layerCount           = 1,
    .colorAttachmentCount = 1,
    .pColorAttachments    = &attachmentInfo
};
```

* `renderArea` is split into two parts:
	* `extent` describes the width and height of the `renderArea` 
	* `offset` describes the x and y offsets of the `renderArea`
* `layerCount` describes the number of [[Vulkan Image Layers|layers]] to render to. This is 1 on a non layered image.
* `colorAttachmentCount` and `pColorAttachments` are the length and data of the array of attachments we have. With only one we just pass the memory address to our `attachmentInfo` object.