# What is it?
> Allocating from the heap is expensive and doing it many times is many times more expensive. So, what if instead of each [[Command Buffers|command buffer]] allocating from the heap whenever it gets created it takes a slice of a pre-allocated pool?

This is the core intuition behind [[Vulkan Command Pools|Command Pools]], they are effectively responsible for actually allocating our command buffers as well as providing some initialization. 
## How to make in Vulkan

```cpp
vk::CommandPoolCreateInfo poolInfo {
	.flags            = vk::CommandPoolCreateFlagBits::eResetCommandBuffer,
	.queueFamilyIndex = queueIndex
};

commandPool = vk::raii::CommandPool(device, poolInfo);
```

* `flags` is a [[Bitwise]] field that enables certain functionality in the pool. 
* `queueFamilyIndex` specifies what graphics family these commands fall under which is important as it determines what operations you can perform. This should almost always be the logical device's QFP.
# Flags

![[Vulkan Command Pool Flags]]