https://zhuanlan.zhihu.com/p/493139981?utm_source=wechat_session&utm_medium=social&utm_oi=804515204485488640&utm_campaign=shareopn







## **Pilot源码速览**

- **Engine**

- - 引擎类PilotEngine的定义及生命周期管理，各个模块的初始化及关闭顺序，逻辑和渲染的Tick入口
  - 渲染上使用了三重缓冲，实现了简易的场景资源管理



- **Core** 引擎核心层，上层功能通用模块

- - **Base**

  - - **marco.h**
      定义常用全局宏，包括Log，sleep，invalid_id等
    - **public_singleton.h**
      单例模板类，调用时静态初始化保证顺序，禁用构造及复制构造函数保证单例

  - **Log**

  - - **log_system.h**
      日志类，继承单例基类，使用spdlog，分五个output等级

  - **Math** 数学库，没有太特殊的内容，但函数注释程度好评

  - - **math_headers.h**
      include用warpper
    - **vector2/3/4** 向量
      常规定义
    - **matrix3/4** 矩阵
      常规定义
    - **quaternion.h** 四元数
      四元数欧拉角转换，旋转和插值函数等
    - **transform.h** translate/rotate/scale变换
      仅有定义 vec3 + vec3 + quat
    - **random.h** 随机数
      随机数发生器，种子生成，正态分布等
    - **math_marco.h**
      常用宏，比大小，取符号等
    - **math.h**
      常量字面值及数据结构声明类，包括radian, degree及常用三角函数
    - **bounding_box.h**
      AABB型

  - **Meta**
    反射、序列化及json引入，详细TODO

  - - **reflection**
      反射功能实现，定义了反射用宏CLASS(), STRUCT(), REFLECTION_BODY()及REFLECTION_TYPE()等，风格上基本与UE相同, metadata包括field，array，class三类
      关键类：TypeMetaRegisterinterface
    - **serializer**
      序列化实现，存储格式为json



- **function** 引擎各个功能模块的实现， TODO

- - **animtion**

  - **controller**

  - **framework**

  - - **object**

    - - 在runtime真正的object类型，可以通过Load/Save等接口读写objet及component资源
      - 可以理解为Component的容器，将一系列Component集中在一起管理生命周期

    - **level**

    - - 用map管理各类object，也支持从ObjectRes创建object实例到关卡中

    - **world**

    - - vector管理多个level，指针记录当前激活的关卡，目前首个load的关卡就是当前激活的关卡

    - **Component**

    - - animtion

      - - 指认skeleton及ComponentRes资源，tick中计算animation pose

      - camera

      - - 指认相机上前左三向量确定视口方向，tick中根据input计算的数值对相机进行更新

      - mesh

      - - 指认多个mesh资源，根据attach的object的transform与animation进行更新
        - raw_mesh 定义了基础primitive类型，包括point,line, triangle, quad，以及indice与vectex buffer格式

      - motor

      - - 指认移动方向与速率，tick时根据输入(键鼠方向及幅度输入)计算当前帧的object目标位置

      - rigidbody

      - - 指认PhysicsActor，对object使用physicsActor进行物理模拟

      - transform

      - - 使用双缓冲记录相邻两帧的transform数据，通过swap进行更新，比较有特点，暂时没看出来目的

  - **input**

  - - **input_system**

    - - 输入系统，包括键位的定义与action的映射，使用了GLFW提供的输入基础定义，在editorMode与GameMode分别有着不同的定义，轴向的转动用视口的光标移动与相机fov进行计算
      - ***想要了解编辑器的基本操作需要看这里的键位定义，较为蛋疼的点是暂时没有做撤销***

  - **physics**

  - **render**

  - **scene**

  - - *******scene_object**

    - - 关键定义: GameObjectComponentDesc

      - - Component基类
        - 包括引用的Mesh, 材质，世界空间transform，动画绑定及动画输出Pose

      - **GameObjectDesc**

      - - 场景中的每个需要渲染和计算动画的GameObject
        - 每个GO通过Id进行index，是一系列GOC(GameObjectComponent)的组合
        - 这里就是非常典型的ECS了，全部通过Id实现，绕开了指针，不过没有看到遍历的实现

      - **ComponentId**

      - - 对于每个GameComponent，都会attach到GameObject上去为其实现功能
        - 每个ComponentId，包括自身的id及其所在GameObject的id

    - **scene_manager** 场景资源管理类

    - - 通过一个Map管理Mesh与GO间的Id映射，应该是为了实现Go复用Mesh的实例化
      - 材质，顶点数据，骨骼，贴图等资源的管理，同样通过map，但是可以双向index，应该是为了避免重复
      - 实现了资源的多线程处理，资源释放会存在类似于pendingkill的队列中集中处理，可能是为gc做准备
      - 尽管叫做sceneManager，但其实只内含一个scene资源，包括setFOV这种函数也直接放到了类中，应该是前期为了方便QwQ

    - **scene_allocator**

    - - 场景资源分配模板类
      - 同样通过两个map管理全局数据，实现了资源element和guid的双向索引

  - **ui**

  - - **UIManager**

    - - 暂时没有什么内容



- **platform**

- - **file_service**

  - - 文件读取的简单封装

  - **path**

  - - 跨平台等文件路径处理，包括获取相对路径，路径拆分，获取文件拓展名、文件名



- **resource**

- - **asset_manager**

  - - **AssetManager**

    - - 实现通过路径对资源进行序列化反序列化，Component的regist部分像是为ECS预留

  - **config_manager**

  - - **config_manager**

    - - 文件夹及资源路径配置定义与接口

  - **res_type** 资源类型，类似于UE蓝图类，需要与对象的类型作区分

  - - **common**

    - - **object.h**

      - - **ComponentDefinitionRes** Component资源定义
        - **ObjectDefinitionRes** Object资源定义，Components的集合
        - **ObjectInstanceRes** Object实例资源
        - 基本与ue相同，但直接在component中包含了transform字段，没有通过组件实现

      - **level.h**

      - - object的集合

      - **world.h**

      - - level的集合

    - **components**

    - - **animtion.h**

      - - 动画功能的实现，每个骨骼节点由index与transform构成，骨架由骨骼数组构成，component记录了每个骨骼节点的当前transform，即当前pose

      - **camera.h**

      - - 作为渲染视口的功能实现，支持FOV调节，分为1P与3P和独立相机三类

      - **mesh.h**

      - - 被渲染的功能，SubMesh的集合，subMesh包括模型资源, transform及材质，类似于ue中mesh一个slot，只不过各自位置独立

      - **motor.h**

      - - 移动功能的实现

      - **rigid_body.h**

      - - 物理碰撞功能的实现，包括sphere, box, capsule三种类型，都具有global与local的transform
        - 另外有RigidBodyActorRes类记录全部带碰撞的actor及其质量和碰撞类型

    - **data**

    - - **animation_clip.h**

      - - AnimNodeMap定义了动画节点结构，AnimationChannel为每帧的骨骼transfrom，AnimationClip包括帧数，动画节点数和一组动画帧，AnimationAsset由骨架文件，一组动画节点和一个对应等AnimationClip组成

      - **material.h**

      - - 材质资源定义, 包括baseColor, metalicRoughness, Normal, Occlusion, Emissive五张贴图资源，正常的PBR实现

      - **mesh_data.h**

      - - mesh顶点定义，包括局部空间坐标，法线方向，切线方向，uv空间坐标
        - 骨骼绑定，支持顶点绑定四个骨骼分别给以不同权重
        - mesh_data定义：顶点，index和binding各自的array

      - **skeleton_data.h**

      - - RawBone 单个骨骼，包括骨骼名，index，父节点index，binding_pose和tpose变换矩阵
        - SkeletonData 骨架数据，记录根骨index，骨骼数组及标记位

      - **skeleton_mask.h**

      - - BoneBlendMask 动画Mask，支持逐骨骼的选择是否计算动画