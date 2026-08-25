# What is a Command Buffer Inheritance Info
> The primary use of this struct is to give Secondary [[Command Buffers]] access to the same information that Primary Command Buffers have access to. Since Secondary buffers are akin to libraries they lack the similar sense of state that Primary buffers have.

```cpp
struct CommandBufferInheritanceInfo { 
	vk::RenderPass renderPass;
	uint32_t subpass; 
	vk::Framebuffer framebuffer; 
	vk::Bool32 occlusionQueryEnable; 
	vk::QueryControlFlags queryFlags; 
	vk::QueryPipelineStatisticFlags pipelineStatistics; 
};
```

- `renderPass` is a handle to a [[Render Pass]] object.
- `subpass` specifies what sub-pass this command buffer will run in.
- `framebuffer` is a handle to a specific [[Framebuffer]] (optional)
- `occlusionQueryEnable` says "can this buffer run if the primary buffer is resolving an occlusion query."
- `queryFlags` is bitmask wrapper defining how occlusion query results are handled.
- `pipelineStatistics` says what Pipeline Statistics can be tracked if the main buffer has an active stats query going. 