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

Vulkan的校验层是一个很重要的东西，在刚开始学的时候经常会不重视，后面遇到问题又返回来看，所以最好一开始就把他弄懂。

首先使用vkEnumerateInstanceLayerProperties去查询所有可用的校验层列表，然后在创建Vulkan实例的时候启动校验层：

![image](https://user-images.githubusercontent.com/56297955/234715250-78e6cdbb-5565-4484-ad51-e80c4dfdc6eb.png)

![image](https://user-images.githubusercontent.com/56297955/234715312-e20a8c8a-ff64-4eb3-86c6-02839a2c953c.png)

### 物理设备和队列族 Physical devices and queue families

物理设备，主要指显卡，在使用Vulkan API时，需要首先枚举和选择可用的物理设备，并查询其支持的Vulkan功能和属性。

定义物理设备，用来储存后面我们找出来的符合条件的物理设备：

![image](https://user-images.githubusercontent.com/56297955/234096619-fa29a175-bd13-482c-98f6-8cc5eb438373.png)

检测所有可用的物理设备，然后保存在vector里面，然后依次检测可用性，把可用的那一个保存在刚刚定义的physicalDevice里面。

![image](https://user-images.githubusercontent.com/56297955/234101437-5c13e9d6-d5ff-42ef-8871-31d447dbb27d.png)

关于vulkan的队列簇，vulkan的操作都是提交到队列中执行的，每个队列簇都包含多个队列，不同的队列支持不同的操作，比如数值计算的队列，图形指令的队列。一个队列簇可以支持多个类型的队列，然后每个类型的队列的个数也可能有多个。比如某个GPU的某一个图形簇，支持图形指令，也支持计算指令，然后支持图形指令的队列有五个，支持计算指令的队列有四个。通常情况下，Vulkan应用程序需要至少一个支持图形操作的队列和一个支持传输操作的队列。这些相同类型的队列可以并行执行，但它们之间可能会存在一些竞争和同步关系，例如，如果多个队列同时访问同一块内存区域，就可能会发生竞争和数据不一致等问题。在使用多个队列时，需要仔细考虑同步和竞争关系，以确保应用程序的正确性和性能。有多少队列簇，每个队列簇包含多少队列，这些就是硬件的特性，不同的GPU硬件设备会有不同的队列簇架构。

在具体使用时，我们对于当前的一个物理设备physicalDevice，使用vkGetPhysicalDeviceQueueFamilyProperties这个API去查询它支持的队列簇，vkGetPhysicalDeviceQueueFamilyProperties的第一个参数就是当前查询的物理设备，第二个参数是支持的队列簇个数，第三个参数是物理设备上支持的所有队列簇的属性信息。

![image](https://user-images.githubusercontent.com/56297955/234163314-db2fb147-7701-4ea8-9e52-1c848f654319.png)

先查队列簇的个数，不写入VkQueueFamilyProperties，然后分配好一块儿这么大的数组，再调用这个函数，这次再把队列簇的信息写入我们定义好的VkQueueFamilyProperties类型的数组里。

![image](https://user-images.githubusercontent.com/56297955/234112795-65ba4f90-5201-4e9a-bab9-16606c9003c8.png)

最后，就可以依次遍历这个数组中的所有队列簇，每个队列簇的信息都由VkQueueFamilyProperties来保存

![image](https://user-images.githubusercontent.com/56297955/234164478-bb6edda9-f3e7-4300-9b10-518709a7ecc0.png)

然后就可以用queueFlags去判断这个队列簇支持哪些队列，它是一个掩码，所以可以与其他掩码做与运算判断支持哪个队列，下面是一些常见的掩码：

![image](https://user-images.githubusercontent.com/56297955/234165049-53ea6201-0e21-4ecd-9434-602ec6f8367d.png)

![image](https://user-images.githubusercontent.com/56297955/234113278-26bb0730-577e-447c-8704-941bd1e262ed.png)

### 逻辑设备和队列

逻辑设备（Logical Device）是与物理设备（Physical Device）关联的一个对象，用于执行命令和管理资源，是应用程序与硬件之间的接口。先创建好物理设备之后，就可以创建逻辑设备，可以说逻辑设备是在物理设备的基础上创建的，它可以包含多个命令队列和多个扩展和功能，以满足应用程序的需求。

**总结一下我自己的理解**

vulkan中设计的物理设备是为了对显卡等硬件（主要是显卡）做一个抽象，用来表示（储存）物理设备的信息（如显卡型号）以及可用的硬件资源（支持的命令队列和硬件可用的特性或拓展）。然后逻辑设备就是在这个的基础上创建的，它是application与hardware之间的接口，可以让应用层通过它去调用硬件资源，而且在创建时可以选择性地包含所需的命令队列和扩展，并指定所需的特性和属性，所以可以认为物理设备提供了一大堆东西，然后逻辑设备可以选择性地使用我们需要的。

PS：

硬件扩展（Hardware Extension）是指Vulkan API中提供的额外功能和特性，这些功能和特性可能并不是所有硬件设备都支持。硬件扩展可以提供一些新的、高级的、专用的或实验性的功能，以满足应用程序的需求。硬件扩展是由GPU厂商提供的，它们可以通过Vulkan API进行查询和启用。在使用硬件扩展之前，需要检查当前物理设备是否支持该扩展。如果支持，可以在创建逻辑设备时启用这些扩展。

硬件扩展可以提供各种不同的功能，例如：

支持新的图形API功能，例如光线追踪、变形反馈等。
提供更好的性能和效率，例如多视口渲染、异步计算等。
支持新硬件特性，例如VR/AR、AI神经网络等。
提供更好的调试和性能分析功能。

**创建逻辑设备的流程**

1、在可用的物理设备选择要使用的物理设备 physicalDevice  
2、填写VkDeviceCreateInfo 结构体，它包含了创建逻辑设备所需的命令队列类型、扩展和特性等信息  
3、使用vkGetDeviceQueue函数获取命令队列句柄    
4、使用vkCreateDevice创建逻辑设备

找到physicalDevice支持的队列：

![image](https://user-images.githubusercontent.com/56297955/234130776-c3abe6a5-0834-44c8-9654-6d6bd99dbc4c.png)

![image](https://user-images.githubusercontent.com/56297955/234130830-d67ec181-feb0-4149-8175-5e8c258fb84f.png)

填写VkDeviceCreateInfo结构体中的VkDeviceQueueCreateInfo，通过它来指定要使用的队列：

![image](https://user-images.githubusercontent.com/56297955/234134473-4a59cde0-31fc-4d5a-b6ed-6d466f0f16d2.png)

创建VkDeviceCreateInfo结构体中的VkDeviceFeatures：

![image](https://user-images.githubusercontent.com/56297955/234134649-11b1b94a-dc1e-46fb-ad42-1ade84dae404.png)

这两个结构写好之后，就来填VkDeviceCreateInfo了，将VkDeviceCreateInfo结构体的pQueueCreateInfos指针指向queueCreateInfo的地址，pEnabledFeatures指针指向deviceFeatures的地址：

![image](https://user-images.githubusercontent.com/56297955/234135267-016ad8b1-e1f4-4fa0-87a3-78ea7444de72.png)

最后就是使用vkCreateDevice来创建逻辑设备，它的参数是VkDeviceCreateInfo，以及确定好的physicalDevice

![image](https://user-images.githubusercontent.com/56297955/234140462-245b1de2-a247-4921-b1b4-bbf7173b29a8.png)![image](https://user-images.githubusercontent.com/56297955/234140800-6e7147c7-878d-48e1-89f3-17f8552f51f6.png)

创建好逻辑设备之后，还需要使用vkGetDeviceQueue去取得创建好的设备的队列簇

![image](https://user-images.githubusercontent.com/56297955/234301885-22625f76-192e-42a6-8577-14a6ab13e95b.png)

![image](https://user-images.githubusercontent.com/56297955/234301958-8cf2a729-2593-4757-8e5b-a617a847915f.png)

**到目前为止，总结一下Vulkan的初始化思路：**

![image](https://user-images.githubusercontent.com/56297955/234165789-ea10189d-41ee-4c60-9717-82f67400c1d6.png)

### 窗口表面&交换链 Window surface & Swapchain

Window Surface 是一个用于将 Vulkan 渲染结果显示在窗口系统中的对象。因为Vulkan是跨平台的，他不能和窗口系统交互，而Window Surface就提供了窗口系统所需的信息，使 Vulkan 能够将渲染结果显示在正确的窗口中。可以用VK_KHR_surface扩展实现。（Khronos为了实现跨平台的目标，为不同平台创造了一个统一的抽象层，名为surface，它是将Vulkan和具体设备显示连接起来的一个桥梁），创建成功后，可以在 Vulkan 中使用 Window Surface层来创建 Swapchain（交换链）。

Vulkan不存在默认的帧缓冲的概念，它需要一个能够缓冲渲染操作的组件。交换链本质上是一个包含了若干等待呈现的图像的队列，应用程序从交换链获取一张图像，在图像上进行渲染操作，完成后将图像返回到交换链队列。所以它本质上就是一个图像缓冲区，每个缓冲区都包含一个完整的图像帧，交换链的缓冲区数量由硬件决定。

ps：

一个物理设备可以有多个交换链，每个交换链进行不同的配置，如专门的缓冲区数量、呈现模式、表面格式等，然后在渲染时使用不同的交换链来优化渲染性能和质量。

首先，我们要再定义一个presentFamily，因为我们需要查找支持图像呈现的队列族的索引：

![image](https://user-images.githubusercontent.com/56297955/234267803-4c31aa2a-882f-4885-ba83-dff58879e6d7.png)

然后使用vkGetPhysicalDeviceSurfaceSupportKHR函数来检查物理设备是否具有图像表现能力，这个在检查是否支持图形指令的地方一起检查就行：

![image](https://user-images.githubusercontent.com/56297955/234296753-b85d3d27-f6db-411a-90fb-9031a6386044.png)

![image](https://user-images.githubusercontent.com/56297955/234298754-f64b3d95-72ca-4faf-a8ba-5271903f3047.png)

在创建交换链时，同样也要检测设备是否支持交换链，和检查物理设备的时候一样，使用类似的vkEnumerateDeviceExtensionProperties，去枚举所有拓展，然后看是否支持

![image](https://user-images.githubusercontent.com/56297955/234308705-0137bf10-4add-4228-84da-e090cf35768d.png)

![image](https://user-images.githubusercontent.com/56297955/234310737-2dd5982b-b8f8-4bb1-87ff-a4de4b443f64.png)

在确定了是否支持之后，就是查询更多的信息，比起创建实例，创建交换链还需要设置更多的细节。Vulkan中会使用SwapChainSupportDetails这个结构体去保存细节信息（自己定义）：

![image](https://user-images.githubusercontent.com/56297955/234342931-9b4e6a5b-94b3-4313-ae57-d810a8d64671.png)

其中，VkSurfaceCapabilitiesKHR是表面特性，它包含交换链可以支持的图像的最小和最大个数，交换链中图像的最小和最大尺寸，交换链中图像支持的像素格式等；VkSurfaceFormatKHR是指像素格式、颜色空间等；VkPresentModeKHR是指交换链在把图像呈现在屏幕上的方式，比如立即模式，渲染出图像不缓存，直接输出到屏幕，另外一种是FIFO，先进先出模式，先保存在交换链缓存里的先渲染到屏幕。这三个就是一个交换链必须要有的能力（特性）。

然后分别使用vkGetPhysicalDeviceSurfaceCapabilitiesKHR、vkGetPhysicalDeviceSurfaceFormatsKHR、vkGetPhysicalDeviceSurfacePresentModesKHR去查询就好了，查询结果保存在结构体里，这个函数的参数是物理设备，查询物理设备的交换链支持：

![image](https://user-images.githubusercontent.com/56297955/234357623-c341e7a2-0079-46b8-9307-98cbd76cbf17.png)


查询完之后依次检测formats、presentModes是不是empty就行了。

确定了这三个基本能力、特性之后，就可以进行下一步，为这三个特性选择合适的设置，比如具体选择哪一个呈现模式，选择哪个颜色空间等。具体代码就不贴了，这就是纯粹地用API。

呈现格式、表面格式和交换链中的图像分辨率都设置好了之后，就可以正式的创建交换链。

首先设置好三个基本能力：

![image](https://user-images.githubusercontent.com/56297955/234425464-057d656e-1b24-4de8-a1e4-0556b69f67de.png)

然后设置好交换链支持的图像个数：

![image](https://user-images.githubusercontent.com/56297955/234425620-3795692d-a66a-4d81-8f88-dd780aca9a57.png)

然后就是填写VkSwapchainCreateInfoKHR结构体：

![image](https://user-images.githubusercontent.com/56297955/234431719-d3691186-4dae-4a1b-9550-e3ff6c7355d7.png)

填完之后就是常规套路，使用API去创建交换链：

![image](https://user-images.githubusercontent.com/56297955/234431991-65e23669-0560-4f60-8984-58081bee84c1.png)

![image](https://user-images.githubusercontent.com/56297955/234432364-7de48fab-7dd7-49f4-a70e-a71c2f003954.png)

创建完之后就可以使用vkGetSwapchainImagesKHR去取得交换链缓冲区的图像，同样是先获取数量，然后再分配数组空间，把图像存在vector里：

![image](https://user-images.githubusercontent.com/56297955/234432265-71ec7c15-d8c4-4e0d-9bc8-b4929f952a12.png)

### 图像视图

在Vulkan中，图像视图（Image View）是用于访问图像（Image）数据的对象，它描述了图像的格式、范围、用途等属性。通过创建图像视图，我们可以将图像数据绑定到渲染管线的特定阶段，如顶点着色器、片段着色器、几何着色器等，以便进行渲染操作。简单来说Image就是一个rendertarget对象，管线绑定上之后就可以访问它。

VkImage就是一个视图对象，需要使用VkImageView绑定之后去访问它，所以首先定义一个vector，去存交换链中的视图：

![image](https://user-images.githubusercontent.com/56297955/234705798-c800c0d2-f5fc-4459-ba52-50c390f3655c.png)

然后遍历交换链中的所有图像，依次填写VkImageViewCreateInfo结构体，给他们创建图像视图：

![image](https://user-images.githubusercontent.com/56297955/234707954-7156700e-ae5a-4bf1-a4aa-f77bce28ee8b.png)

![image](https://user-images.githubusercontent.com/56297955/234712134-96ca0981-6771-45b6-8d59-5bee1b369e05.png)

以上就是一些基础，大多是查API，然后填空，流程也比较固定，下面才是经常会用到的重点。

### Graphics pipeline basics

大部分都和常规的图形管线一样，但Vulkan不是很支持管线的动态设置，都是在管线创建前提前设置好的

![image](https://user-images.githubusercontent.com/56297955/234718193-d7bb875d-a3f0-4a19-9de9-e0467fee21bc.png)

首先创建一个createGraphicsPipeline函数，然后在里面去设置我们的渲染管线，在设置好图像视图之后调用：

![image](https://user-images.githubusercontent.com/56297955/234721934-ef7f2368-6146-4567-bac7-b13397255d28.png)

在Vulkan中shader有着点点不一样，它的着色器代码需要被编译为SPIR-V字节码格式，然后才能被Vulkan API加载和使用。使用glslc.exe -o去编译。

编译好之后去读取这个字节码的文件：

![image](https://user-images.githubusercontent.com/56297955/234999867-ca385050-a82a-40e6-98db-83395464a1d2.png)

要将着色器字节码在管线上使用，还需要使用VkShaderModule对象，创建着色器模块也是以一样的思路，去填写必要的结构体，首先shader模块是由vkCreateShaderModule这个api创建的，这个API需要设置几个参数：

![image](https://user-images.githubusercontent.com/56297955/235067932-c5c8e288-09dd-492f-9c3f-0e6a869d9158.png)

首先是设置第一个参数，VkShaderModuleCreateInfo，设置一下它的结构体类型，着色器代码字节数等等：

![image](https://user-images.githubusercontent.com/56297955/235068660-7a63f008-7e42-4b97-86a6-d5178403bb12.png)

着色器模块对象只在管线创建时需要，所以，不需要将它定义为一个成员变量，我们将它作为一个局部变量定义在createGraphicsPipeline函数中：

![image](https://user-images.githubusercontent.com/56297955/235068760-fa9cdc1e-d763-46e5-82a8-d9340373a44d.png)























































