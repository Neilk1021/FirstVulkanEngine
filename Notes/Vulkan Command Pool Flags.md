# Transient
> `vk::CommandPoolCreateFlagBits::eTransient` is a [[Bitwise]] flag that says "hey these command buffers are going to die really quick, you can allocate it kind of poorly because its going to be gone very soon."

By default allocating memory can be very slow as the CPU needs to find a "right sized" chunk of memory. This starts fast but degrades over time if we're constantly allocating and deallocating memory from our [[Vulkan Command Pools|Command Pool]]. This is because the memory starts looking more similar to Swiss cheese. 

What `vk::CommandPoolCreateFlagBits::eTransient` does is tell the allocator to switch to a faster method that effectively appends the allocation onto the end (similar to how stack allocation works). The catch is that it does not care about [[Memory Fragmentation|memory fragmentation]].
# Reset
>`vk::CommandPoolCreateFlagBits::eResetCommandBuffer` is a [[Bitwise]] flag that says "Hey I would like to be able to clear and replace individual command buffers at runtime please." 

By default Vulkan doesn't allow this because it's slower. If I need to clear  just one thing then I need to keep track of that, but if I clear everything I can just move a pointer and be done.  When we set this flag to true we can call ``vkResetCommandBuffer`` and `vkBeginCommandBuffer`.

# Combination of the two.
> How does Vulkan handle it if we tell it "I want to be able to individually reset [[Command Buffers]] and these command buffers are incredibly short lived." 

The answer is that Vulkan will often just not really delete your command buffer, but instead allocate a new buffer onto the end as its quicker and the entire pool will be flushed soon regardless.