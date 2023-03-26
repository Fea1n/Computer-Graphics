创建项目模板





初始化窗口

初始化帧缓冲

初始化vulkan

主循环

清理交换链

清理



加载模型代码

初始化vulkan 

创建实例

启动debug信使

创建表面

挑选物理设备

创建逻辑设备

创建交换链

创建图像视图

创建渲染通道

创建描述符布局

创建图形管线

创建图形管线

创建命令池

创建深度资源

创建帧缓冲

创建纹理图

创建纹理视图

创建纹理采样器

加载模型

创建顶点缓冲

创建索引缓冲

创建均匀缓冲

创建描述符池

创建描述符集

创建命令缓冲

创建同步对象







顶点着色器对比

创建着色器

struct 

{

}

加入顶点缓冲前/后

```cpp
#version 450

layout(location = 0) out vec3 fragColor;

vec2 positions[3] = vec2[](
    vec2(0.0, -0.5),
    vec2(0.5, 0.5),
    vec2(-0.5, 0.5)
);

vec3 colors[3] = vec3[](
    vec3(1.0, 0.0, 0.0),
    vec3(0.0, 1.0, 0.0),
    vec3(0.0, 0.0, 1.0)
);

void main() {
    gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0);
    fragColor = colors[gl_VertexIndex];
}
```

```cpp
#version 450

layout(location = 0) in vec2 inPosition;
layout(location = 1) in vec3 inColor;

layout(location = 0) out vec3 fragColor;

void main() {
    gl_Position = vec4(inPosition, 0.0, 1.0);
    fragColor = inColor;
}
```





暂存缓冲区



传输队列



抽象缓冲区创建



使用临时缓冲区



集成 imgui 需要做的事情

draw_frame前\

准备imgui所需要的descriptor pool
 创建imgui的context      (`ImGui::CreateContext`)
 绑定具体的surface       (`ImGui_ImplGlfw_InitForVulkan`)
 绑定相应的Vulkan上下文  (`ImGui_ImplVulkan_Init`)
 绑定相应的字体资源      (`ImGui_ImplVulkan_CreateFontsTexture`),需要`command buffer`，并进行`queueSubmit` (清理字体资源)

draw_frame中进行queue_submit之前

```cpp
    //生成Imgui的new frame
        ImGui_ImplVulkan_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame(); 
    //具体的UI生成
        ;
    //render后获取ImDrawData
        ImGui::Render();
        ImDrawData* draw_data = ImGui::GetDrawData();
```

在draw_frame的command buffer中加入

```scss
ImGui_ImplVulkan_RenderDrawData(draw_data,m_command_buffer);// CORE!!
```

draw_frame后的资源清理

```cpp
    vkDestroyDescriptorPool(m_device, imgui_descriptor_pool, m_allocator);
    ImGui_ImplVulkan_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImGui::DestroyContext();
复制代码
```

完成Vulkan+GLFW对DearImgui的集成