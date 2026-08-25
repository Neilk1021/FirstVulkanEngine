# What is the Pipeline Layout?
>The pipeline layout tells the GPU where certain inputs are coming from in regards to shaders. For example "a texture sampler can be found at slot X." This gives the blueprint of how the GPU needs to configure itself for your code to work.

To create an empty blueprint we do
```cpp
vk::PipelineLayoutCreateInfo pipelineLayoutInfo{ 
	.setLayoutCount = 0, 
	.pushConstantRangeCount = 0 
};
```

This says "there are no layout groups (texture Samplers, transformation matrices)" so don't set it up. Afterwards we create the pipeline via the following code:
```cpp
pipelineLayout = vk::raii::PipelineLayout(device, pipelineLayoutInfo);
```