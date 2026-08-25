# What is Color Blending?
>When the [[Fragment Shader]] outputs a color its the Color Blending FF's responsibility to blend that color and the color already existing in the [[Framebuffer]]. This can be done via mixing the new and old color or combining the two via a bitwise operation. 

# Attachment State
Attachment state is a color blending setting attached to an individual [[Framebuffer]]. In Vulkan this is `vk::PipelineColorBlendAttachmentState`. An example set-up is seen here:

```cpp
vk::PipelineColorBlendAttachmentState colorBlendAttachment{
    .blendEnable    = vk::False,
    .colorWriteMask = vk::ColorComponentFlagBits::eR | 
					  vk::ColorComponentFlagBits::eG | 
					  vk::ColorComponentFlagBits::eB | 
					  vk::ColorComponentFlagBits::eA
};
```

This is saying "Do not blend the new and the old color" (that's the blendEnable = false), and write the R,G,B, and A values over the previous [[RGBA]] values.  

Under the hood the operation looks like this.  Where the RGB channels gets blended via `colorBlendOp` and the A channels get blended via the `alphaBlendOp`. It should be noted that both `colorBlendOp` and `alphaBlendOp` are set in the `PipelineColorBlendAttachmentState` struct.

```cpp
if (blendEnable) {
    finalColor.rgb = 
	    (srcColorBlendFactor * newColor.rgb) <colorBlendOp> 
	    (dstColorBlendFactor * oldColor.rgb);
	    
    finalColor.a = (srcAlphaBlendFactor * newColor.a) <alphaBlendOp> 
				   (dstAlphaBlendFactor * oldColor.a);			   
} 
else {
    finalColor = newColor;
}

finalColor = finalColor & colorWriteMask;
```

Only colors white-flagged in the `colorWriteMask` will be passed through.

The most common use of this is to implement [[Alpha Blending]] like so:

```cpp
finalColor.rgb = newAlpha * newColor + (1 - newAlpha) * oldColor;
finalColor.a = newAlpha.a;
```

Our default `vk::PipelineColorBlendAttachmentState` to enable alpha blending looks like this:

```cpp
vk::PipelineColorBlendAttachmentState colorBlendAttachment{
    .blendEnable         = vk::True,
    .srcColorBlendFactor = vk::BlendFactor::eSrcAlpha,
    .dstColorBlendFactor = vk::BlendFactor::eOneMinusSrcAlpha,
    .colorBlendOp        = vk::BlendOp::eAdd,
    .srcAlphaBlendFactor = vk::BlendFactor::eOne,
    .dstAlphaBlendFactor = vk::BlendFactor::eZero,
    .alphaBlendOp        = vk::BlendOp::eAdd,
    .colorWriteMask      = vk::ColorComponentFlagBits::eR | 
						   vk::ColorComponentFlagBits::eG |
						   vk::ColorComponentFlagBits::eB | 
						   vk::ColorComponentFlagBits::eA
};
```

# State Create Info
This is the global color blending configuration and tells us how many [[#Attachment State|Attachment States]] we have and what they are. This is also where we can use logical bitwise blending (this is typically things like UI or masking). This also acts as an override as any individual attachments will not do math color blending if `logicOp` is enabled. 

```cpp
vk::PipelineColorBlendStateCreateInfo colorBlending{
    .logicOpEnable = vk::False, 
    .logicOp = vk::LogicOp::eCopy, 
    .attachmentCount = 1, 
    .pAttachments = &colorBlendAttachment
};
```
