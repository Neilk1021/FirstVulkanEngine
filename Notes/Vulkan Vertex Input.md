# What is a Vertex Input?
>The vertex input in Vulkan is a [[Fixed Functions|Fixed Function]] with the primary goal of figuring out **"Where am I getting all of my vertices from"** and **"What do my vertices look like?"**
# How?
Vertex inputs splits this responsibility into two structs `VkVertexInputBindingDescription` and `VkVertexInputAttributeDescription` which I will refer to as bindings and attributes respectively. 
## Bindings (Where is my data)
>This describes **how** we pull the data and **where** exactly we pull it from.

*The struct is defined as follows:*
```c++
typedef struct VkVertexInputBindingDescription { 
	uint32_t binding; 
	uint32_t stride; 
	VkVertexInputRate inputRate; 
} VkVertexInputBindingDescription;
```
### What is that binding thing?
> `binding` is a **unique** index that we assign our middleman and is basically saying "if you send something to this index I want it to follow these rules" 

A good question is *why* do we do this, and the answer is when we're writing shader code we can specify different buffers to pull from. A color's data might look very different than a position therefore these different bindings allow us to correlate different types of data with different buffers.
```glsl
// Look inside the buffer bound to Binding 0
layout(location = 0) in vec3 inPosition;

// Look inside the buffer bound to Binding 2 (location != binding)
layout(location = 1) in vec3 inColor;    
```
### Stride & Input Rate
-  `stride` is the size (in bytes) of one "vertex" when reading our stream of data. 
- `inputRate` is when to "reload" our buffer. 
	- `VK_VERTEX_INPUT_RATE_VERTEX` is standard and basically says "once you've drawn a vertex move the read pointer forward."
	- `VK_VERTEX_INPUT_RATE_INSTANCE` is only move to the next element when an entirely new instance is draw. Think like you have a bunch of cubes and every cube only has one color that all the vertices share. You'd only want to flush the color once every vertex has been placed.

## Attributes (What is my data)
>This describes **what**  Vulkan is looking at when its reading the vertex data.

*The struct is defined as follows:*
```c++
typedef struct VkVertexInputAttributeDescription {
    uint32_t    location;
    uint32_t    binding;
    VkFormat    format;
    uint32_t    offset;
} VkVertexInputAttributeDescription;
```
### Member definitions
* `location`: 
	  When I write GLSL and specify `location = 0` I want you to load up this attribute.
* `binding`
	  You can find the data for this attribute in this binding e.g. `binding = 1`. 
* `format`
	  When you go into the binding you are expecting this datatype e.g. `VK_FORMAT_R32G32B32_SFLOAT`
- `offset`
	  In the binding you should start reading at this many bytes away from the start of the vertex.

# Handing it to Vulkan
> How do we even tell Vulkan to do this? With everything in Vulkan it's a struct.

```c++
VkPipelineVertexInputStateCreateInfo vertexInputInfo{
	.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO,
	.vertexBindingDescriptionCount = 1,
	.pVertexBindingDescriptions = &bindingDescription,
	.vertexAttributeDescriptionCount = static_cast<uint32_t>(attributeDescriptions.size()),
	.pVertexAttributeDescriptions = attributeDescriptions.data(),
}
```
