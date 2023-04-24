Vulkan官方教程:

https://vulkan-tutorial.com/Introduction

记录一下对Vulkan的基础学习

### Vulkan实例 Instance  

instance是Vulkan application和Vulkan library之间的一个connection，用来管理和维护Vulkan的全局信息。具体创建时需要填写VkInstanceCreateInfo结构体：

![image](https://user-images.githubusercontent.com/56297955/233992126-8a8c7ba3-ac1b-495b-97e2-b49b009169a9.png)

![image](https://user-images.githubusercontent.com/56297955/233993003-8068b364-2be8-41f9-9092-ba4016ede589.png)

第一个参数就是VkInstanceCreateInfo结构体，这样就创建好了一个实例，一般只创建一个实例。


### 校验层 Validation Layers

是一组可选的组件，用于在开发和调试过程中检测和报告Vulkan API调用中的错误和潜在问题。校验层可以检测一些常见的错误，例如内存泄漏、资源管理问题、未初始化的变量等等，并提供了有用的调试和错误信息。

### 物理设备和队列族 Physical devices and queue families

物理设备，主要指显卡，在使用Vulkan API时，需要首先枚举和选择可用的物理设备，并查询其支持的Vulkan功能和属性。

定义物理设备，用来储存后面我们找出来的符合条件的物理设备：

![image](https://user-images.githubusercontent.com/56297955/234096619-fa29a175-bd13-482c-98f6-8cc5eb438373.png)

