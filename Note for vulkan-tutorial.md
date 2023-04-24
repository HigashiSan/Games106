Vulkan官方教程:

https://vulkan-tutorial.com/Introduction

记录一下对Vulkan的基础学习

### Vulkan实例 Instance  

instance是Vulkan application和Vulkan library之间的一个connection，用来管理和维护Vulkan的全局信息。具体创建时需要填写VkInstanceCreateInfo结构体：

![image](https://user-images.githubusercontent.com/56297955/233992126-8a8c7ba3-ac1b-495b-97e2-b49b009169a9.png)

![image](https://user-images.githubusercontent.com/56297955/233993003-8068b364-2be8-41f9-9092-ba4016ede589.png)

第一个参数就是VkInstanceCreateInfo结构体，这样就创建好了一个实例，一般只创建一个实例。


### 校验层 Validation Layers
