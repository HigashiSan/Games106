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

#### ShaderModule

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

在创建好之后，在这个函数函数结束前，需要把这个对象使用vkDestroyShaderModule销毁，因为它只是我们创建pipelinestage时临时需要的一个东西

![image](https://user-images.githubusercontent.com/56297955/235326785-d43be161-2dfb-4891-b924-e710ea6ef3a5.png)

把着色器载入之后，就是去配置VkPipelineShaderStageCreateInfo，它的其中一个参数就是我们刚刚配置的ShaderModule，这个stageInfo用于指定管线中着色器阶段的创建信息：

![image](https://user-images.githubusercontent.com/56297955/235330294-ca19dc81-0504-4cbd-9515-3ae94b0125f3.png)

![image](https://user-images.githubusercontent.com/56297955/235330331-561921da-41ca-4f70-8d05-c6b876872734.png)

在配置完shader之后，就还剩管线的固定部分，现在用一个模块去总结一下pipeline的其它部分，Fixed functions、render passes

#### Fixed functions

在 Vulkan 渲染管线中，固定功能是指渲染管线中不需要自定义着色器的部分，这些功能由硬件实现，并由 Vulkan API 提供标准化的接口。相当于配置一下这些接口，就可以让硬件按照指定的方式渲染。比如：使用VkPipelineVertexInputStateCreateInfo去描述传递给顶点着色器的顶点数据格式；使用VkPipelineInputAssemblyStateCreateInfo描述点数据定义了哪种类型的几何图元，以及是否启用几何图元重启，使用VkViewport 定义视口；使用VkPipelineRasterizationStateCreateInfo结构体对光栅化过程进行配置；使用VkPipelineMultisampleStateCreateInfo进行MSAA的配置；使用VkPipelineDepthStencilStateCreateInfo配置深度测试和模板测试；对于颜色混合，也就是blend，有两种个结构体可以用来配置，VkPipelineColorBlendAttachmentState和VkPipelineColorBlendStateCreateInfo；对于Vulkan的管线，有极少的部分可以动态改变，可以使用VkPipelineDynamicStateCreateInfo进行配置；对于着色器的一些全局的uniform对象，需要在创建管线时使用VkPipelineLayout去指定

#### Render Passes

Render pass是Vulkan中描述渲染操作的机制，可以理解为一种渲染操作的容器。它定义了一组附着点（attachment），并指定了一系列渲染操作，以及这些操作对附着点的影响。

在Render pass中，每个附着点都代表了一个图像视图（image view）以及一组加载和存储操作。通过定义附着点，开发人员可以告诉Vulkan如何使用图像资源，并指定在渲染操作中对图像资源进行的操作，例如清空、加载和存储等。

除了附着点，Render pass还包含了一个或多个子通道（subpass），每个子通道代表了一组渲染操作。在每个子通道中，开发人员可以指定需要渲染的图形管线、使用的附着点以及渲染操作的顺序。子通道之间可以定义依赖关系，以确保渲染操作的正确顺序。

通过将渲染操作组织为Render pass，Vulkan可以在编译时对渲染操作进行优化，以提高渲染效率。例如，Vulkan可以在渲染到附着点之前执行一些预处理操作，以减少需要写入附着点的数据量。此外，Render pass还可以利用Vulkan的多重采样和深度测试等特性，以实现高效的渲染操作。

**总结一下：render pass就是Vulkan用来定义渲染操作的，他就是一系列对帧缓冲的操作，然后操作完成之后如何处理结果，是保存，还是舍弃。**

**在定义renderpass时，可以定义多个attachment，每个attachment都是一个帧缓冲和一组加载和存储操作以及最终的布局方式，然后定义子流程（也就是一系列渲染流程），子流程会指定它依赖的attachment，一些系列的渲染构成一个renderpass，renderpass类似于unity的renderfeature，子流程就类似于每一个提交到contex的cmd，一个feature可以有多个cmd操作**

定义好了前面的shadermodule、管线fixed部分和renderpass之后，就可以具体创建图形管线对象，使用VkGraphicsPipelineCreateInfo这个API，又到了填api的时候：

![image](https://user-images.githubusercontent.com/56297955/235565894-b5d5c0c0-3283-4f44-852c-f444499174a5.png)

sType是Vulkan中用来描述结构体类型的，比如告诉vulkan这个结构体是一个VkGraphicsPipelineCreateInfo结构体，stageCount是我们要使用的可编程着色器个数，使用pStages去接收这些着色器，它是使用VkPipelineShaderStageCreateInfo去描述的。剩下的部分如pVertexInputState这些，就是引用之前填好的结构体。

这个结构体填好之后，就可以创建我们的管线对象了：

![image](https://user-images.githubusercontent.com/56297955/235579495-fcd7c068-98f5-4490-a112-a0752ac0994c.png)

![image](https://user-images.githubusercontent.com/56297955/235578949-5d5db67f-ff95-47b0-8b0a-7535044d2248.png)

管线对象创建好之后，就是为renderpass绑定帧缓冲，帧缓冲的个数就是之前硬件支持的交换链的个数，同样是配置帧缓冲结构体，然后create：

![image](https://user-images.githubusercontent.com/56297955/235597572-ddfae973-56a1-4c7b-be3c-437fa8c9a656.png)

这些所有准备工作设置好之后，就是真正的绘制指令，command buffer，vulkan的命令也是提交到pool里，然后统一对一个pool里的命令进行操作，这些东西和unity里面的pool，cmd大同小异。在管线配置好之后，使用VkCommandPool创建一个命令池对象，填写完命令池的结构体之后使用vkCreateCommandPool创建命令池：

![image](https://user-images.githubusercontent.com/56297955/235600639-4ceffaed-a07d-44f1-9ac8-d799d1756ec9.png)

然后就是创建cmd，

![image](https://user-images.githubusercontent.com/56297955/235603907-6075784a-04f9-4db7-ae39-94ecae7f9047.png)

![image](https://user-images.githubusercontent.com/56297955/235604075-1d8d03f1-9ba3-4a66-9ca2-9c66862e7205.png)

创建完cmd之后就是使用vkBeginCommandBuffer记录这些命令：

![image](https://user-images.githubusercontent.com/56297955/235604742-e39d3598-9446-486f-96da-83eb0541f7df.png)

然后就是正式开始渲染流程，使用VkRenderPassBeginInfo结构体来指定开始执行渲染流程所需的信息：

![image](https://user-images.githubusercontent.com/56297955/235606323-0b71c46e-ef8b-4ded-adfc-d9dc8dda74e4.png)

然后把这个信息绑定到当前的渲染管线：

![image](https://user-images.githubusercontent.com/56297955/235606416-8aca3f55-5116-4924-b530-28e18f93c6f9.png)

然后就可以调用vkCmdDraw进行绘制：

![image](https://user-images.githubusercontent.com/56297955/235610912-ad2af066-5e25-4781-8060-7e2128ac6d94.png)

最后再结束渲染流程，并记录cmd：

![image](https://user-images.githubusercontent.com/56297955/235611584-5c8f00cc-b205-49fb-b5ea-cd315ec60fd8.png)

具体的渲染指令，就在vkCmdBeginRenderPass和vkCmdEndRenderPass之间。和unity的逻辑一样，在后面会把里面的绘制指令提交，然后自动执行。

### 渲染呈现

在主循环中取得交换链的图像，执行缓冲区的cmd，返回结果图像到交换链，然后呈现。

为了控制先渲染，再呈现结果，如果有结果了，就不渲染，使用VkSemaphore去设置信号，创建两个信号，一个信号量发出图像已经被获取，可以开始渲染的信号；一个信号量发出渲染已经结果，可以开始呈现的信号。从交换链获取图像使用vkAcquireNextImageKHR去实现，创建信号在准备阶段进行，在mainloop的draw之前：

![image](https://user-images.githubusercontent.com/56297955/235735207-fc4f2135-70c9-4de5-957c-ddf966da2496.png)

截至到目前为止，无论是管线创建、信号量创建、cmd创建、cmd绑定到pipeline。。。都是准备工作，下面就是真正提交cmd，提交cmd之后，才能执行

要提交cmd，会使用到VkSubmitInfo结构体。VkSubmitInfo结构体用于指定提交一组Command Buffer时的信息，主要用于指定需要提交的Command Buffer、信号量和等待信号量、等待的管线阶段以及提交的信号量和等待信号量的信号值等信息，以实现Command Buffer之间的同步和确保执行顺序的正确性，所以首先描述VkSubmitInfo：

![image](https://user-images.githubusercontent.com/56297955/235740071-59acc14b-8093-4ee9-ad03-abb0b81a908d.png)

然后使用vkQueueSubmit提交需要提交的cmd：

![image](https://user-images.githubusercontent.com/56297955/235743544-71e9217d-74d7-4b1e-9a73-b162220420fb.png)

vulkan在执行完提交的cmd之后，就需要返回帧缓冲，把图像返回给交换链，呈现到屏幕上，通过VkPresentInfoKHR配置呈现信息，然后通过vkQueuePresentKHR呈现图像

![image](https://user-images.githubusercontent.com/56297955/235789057-40d1f834-673b-4547-ad63-7bc7921ed833.png)

这样就可以画出三角形了，vulkan的基础确实很多，但一两天看完没啥问题，比起opengl确实麻烦了不少，但同时也感觉到了可自己控制的部分也在增多。

### 顶点缓冲 Vertex Buffer

这是我学得很痛苦的一个东西，看了很久，才把所有的疑问解决，然后理解，这里记录一下。

之前的顶点信息是硬编码在顶点着色器里的：

![image](https://user-images.githubusercontent.com/56297955/235811132-d4a804c5-3384-46b3-9581-de6b52abeb65.png)

现在以顶点缓冲的形式往shader里面送数据：

![image](https://user-images.githubusercontent.com/56297955/235811242-dea4c387-5955-406c-8289-45f831e29a85.png)

在对顶点的描述上，需要掌握两个API：VkVertexInputBindingDescription，描述顶点的存放，包括stride，binding索引等，以及VkVertexInputAttributeDescription描述顶点属性，包括顶点数据存放的location、数据类型format等。

分别用两个函数实现：

![image](https://user-images.githubusercontent.com/56297955/235825082-7a186f92-de5a-4c70-9e8f-91b6fff5fda0.png)

然后就是把数据给管线，通过填写VkPipelineVertexInputStateCreateInfo结构体，这个结构体是在创建pipeline的时候创建的：

![image](https://user-images.githubusercontent.com/56297955/235827302-668c83d6-635c-4d4a-ab9d-8095f6724d81.png)

把顶点数据送到管线之后，就创建顶点缓冲，高效地把顶点数据存储在GPU的特定区域，首先同样地填写VkBufferCreateInfo结构体，然后创建：

![image](https://user-images.githubusercontent.com/56297955/235833684-530cf4f1-7d98-46a8-806b-1c29dbf0d1c3.png)

创建好之后为它分配空间，后面我们就可以把顶点数据放在这里，分配空间首先使用VkMemoryRequirements结构体，用vkGetBufferMemoryRequirements查询我们缓冲区对象所需的内存大小和内存类型：

![image](https://user-images.githubusercontent.com/56297955/235854620-fd276b10-54cb-4405-aad4-4de16628d01d.png)

![image](https://user-images.githubusercontent.com/56297955/235856369-8bd03dd1-b2af-4631-8e0b-5c425d83ef92.png)

然后使用vkGetPhysicalDeviceMemoryProperties函数查询物理设备可用的内存类型：

![image](https://user-images.githubusercontent.com/56297955/235858511-6eac0ab6-9f75-4052-86c8-041ddd4ef5cb.png)

这些都准备好之后，就是填写VkMemoryAllocateInfo结构体，然后就可以用它指定内存地分配：

![image](https://user-images.githubusercontent.com/56297955/235859217-48cc8ee7-a85d-4199-b25a-7e87138f4bc5.png)

分配好之后，就可以使用vkBindBufferMemory把这片内存与之前地顶点数据关联：

![image](https://user-images.githubusercontent.com/56297955/235859703-3ecad532-0859-440c-8819-1dbe3c542a01.png)

最后就可以使用vkMapMemory将缓冲对应的那一部分GPU内存映射到CPU的顶点数据所在的那一部分内存，然后用memcpy将数据实际地复制过去：

![image](https://user-images.githubusercontent.com/56297955/235862025-d4eace43-be20-4d1b-9203-8c71c7ec11d5.png)

然后就绑定好缓冲：

![image](https://user-images.githubusercontent.com/56297955/235864949-951bb769-2931-45ce-b50a-1a927c1faaf0.png)

这里的第二个参数就是layout的缓冲区位置，location = 0，如果把第二个参数改成不与location匹配的，就拿不到对应的顶点数据。

**总结一下流程，确定好shader里的layout，明确要取的顶点数据从哪个location拿，然后定义VkBuffer vertexBuffer，创建buffer，在GPU上为它分配内存空间，然后把buffer绑定在这片内存，然后把数据copy到这篇内存。**

### 暂存缓冲 Staging buffer

### 引索缓冲 Index buffer

和opengl几乎一样，创建它的流程也和vertexbuffer差不多

### Uniform buffer

重点。uniform变量，主要是为了可以在程序运行时被修改，也就是Uniform Buffer Object（UBO对象），它主要用于存储Uniform数据的缓冲区对象。UBO对象可以包含多个Uniform变量，这些变量在渲染管线中的Shader Stage中使用。UBO对象的数据可以在应用程序运行时进行修改，并且可以被多个Draw调用共享。

它本质上也是一个buffer，所以创建过程和vertexbuffer差不多：

1、创建Uniform Buffer的数据结构：在应用程序中定义Uniform数据结构，即一个struct。

2、创建Uniform Buffer的VkBuffer对象：使用VkBufferCreateInfo结构体创建一个Uniform Buffer的VkBuffer对象。

3、分配Uniform Buffer的内存：使用VkMemoryRequirements和VkDeviceMemory对象来分配Uniform Buffer的内存。

4、将Uniform Buffer的数据映射到CPU内存：使用vkMapMemory函数将Uniform Buffer的数据映射到CPU内存中，以便CPU可以对Uniform数据进行修改。

5、将Uniform数据复制到Uniform Buffer中：在CPU内存中修改Uniform数据，并使用vkMemcpy函数将数据从CPU内存复制到Uniform Buffer中。

6、将Uniform Buffer绑定到渲染管线中的Shader Stage上：使用vkCmdBindDescriptorSets函数将Uniform Buffer对象绑定到渲染管线中的Shader Stage上。

于是一个UBO对象（我们需要的数据集合）就创建好了，注意shader中的layout与struct定义的数据要一致，这样就可以在shader里面通过layout去访问变量或者资源。

layout就是描述符，set就是符级，一个set可以有多个描述符。


































