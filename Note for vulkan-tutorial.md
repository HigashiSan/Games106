Vulkan官方教程:

https://vulkan-tutorial.com/Introduction

记录一下对Vulkan的基础学习

### Vulkan实例 Instance  

instance是Vulkan application和Vulkan library之间的一个connection，用来管理和维护Vulkan的全局信息。  具体创建时使用VkInstanceCreateInfo创建一个结构体，然后填写这个结构体的信息（可选），包括应用程序名称、版本等。


### 校验层 Validation Layers
