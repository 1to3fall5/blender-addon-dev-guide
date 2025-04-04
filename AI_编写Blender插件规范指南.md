# AI编写Blender插件规范指南

## 目录

- [基本概念](#基本概念)
- [开发环境设置](#开发环境设置)
- [插件结构](#插件结构)
- [命名规范](#命名规范)
- [文件夹组织规范](#文件夹组织规范)
- [关键组件](#关键组件)
- [API使用规范](#api使用规范)
- [界面设计](#界面设计)
- [UI设计可视化](#ui设计可视化)
- [性能优化](#性能优化)
- [测试与调试](#测试与调试)
- [文档规范](#文档规范)
- [代码移植与重构](#代码移植与重构)
- [发布与分发](#发布与分发)
- [常见问题解决](#常见问题解决)
- [示例模板](#示例模板)

## 基本概念

### Blender插件概述

Blender插件（Add-on）是扩展Blender功能的Python脚本集合。AI在编写Blender插件时需要了解以下基本概念：

- **Blender版本兼容性**：明确支持的Blender版本范围，通常在`bl_info`字典中定义
- **Python版本**：Blender内置的Python版本（如Blender 2.93使用Python 3.9）
- **插件类型**：工具类（Tool）、物体生成类（Object）、材质类（Material）等
- **API层级**：bpy、bmesh、bgl等不同模块的作用域和用途

### 插件分类

- **官方插件**：Blender自带的插件
- **社区插件**：第三方开发者创建的插件
- **商业插件**：付费使用的高级功能插件

## 开发环境设置

### 推荐的开发环境

- **编辑器/IDE**：Visual Studio Code + Blender开发插件
- **调试工具**：Blender自带的Python控制台、Visual Studio Code的调试器
- **版本控制**：Git（推荐使用.gitignore排除__pycache__等文件）

### 开发环境配置步骤

1. **安装Blender**：下载并安装目标版本的Blender
2. **设置外部编辑器**：配置Blender使用外部编辑器编写代码
3. **配置插件路径**：找到Blender的插件目录（通常在`用户/AppData/Roaming/Blender Foundation/Blender/版本号/scripts/addons`）
4. **启用开发者选项**：在Blender首选项中启用插件开发相关选项

## 插件结构

### 基本文件结构

```
my_addon/
│
├── __init__.py       # 主要插件文件，包含bl_info和注册/注销函数
├── operators.py      # 操作类定义
├── panels.py         # 界面面板定义
├── properties.py     # 属性定义
├── functions.py      # 核心功能实现
│
├── icons/            # 图标资源
├── presets/          # 预设文件
└── lib/              # 第三方库或依赖
```

### 必要组件

- **bl_info**：插件元数据字典，定义名称、版本、作者等信息
- **类注册和注销**：`register()`和`unregister()`函数
- **操作符（Operator）**：继承自`bpy.types.Operator`的功能类
- **面板（Panel）**：继承自`bpy.types.Panel`的界面类

### bl_info字典示例

```python
bl_info = {
    "name": "插件名称",
    "author": "作者名",
    "version": (1, 0, 0),
    "blender": (2, 93, 0),
    "location": "视图3D > 侧边栏 > 我的插件",
    "description": "插件功能描述",
    "warning": "警告信息",
    "doc_url": "文档URL",
    "category": "插件类别",
}
```

## 命名规范

### 插件命名

- **插件文件夹名**：使用小写字母和下划线，如`my_addon`、`material_library`
- **插件包名**：与文件夹名一致，遵循Python包命名规则
- **插件显示名**：在`bl_info`中定义，可使用空格和大写，如"Material Library"

### 类命名

- **操作符类**：使用`大写字母_OT_功能名`格式，如`OBJECT_OT_add_cube`
  - `OT`表示Operator Type
  - 前缀表示所属类别（如`OBJECT`、`MESH`、`MATERIAL`）
- **面板类**：使用`空间类型_PT_功能名`格式，如`VIEW3D_PT_tools_panel`
  - `PT`表示Panel Type
  - 前缀表示所属空间（如`VIEW3D`、`PROPERTIES`、`NODE_EDITOR`）
- **属性组类**：使用大驼峰命名法（PascalCase），如`MyAddonProperties`
- **其他工具类**：使用大驼峰命名法，如`MeshGenerator`、`TextureManager`

### 标识符命名

- **操作符ID**：使用`类别.操作名`格式，如`object.add_cube`、`mesh.remove_doubles`
  - 类别部分使用小写，表示操作的范围
  - 操作名使用小写和下划线，清晰描述功能
- **属性ID**：使用小写下划线命名法（snake_case），如`my_float_value`、`use_smooth_shading`
- **枚举值**：使用大写下划线格式，如`MODE_EDIT`、`DISPLAY_WIREFRAME`

### 函数和变量命名

- **函数名**：使用小写下划线命名法，如`create_mesh()`、`apply_material()`
- **私有函数**：使用前导下划线，如`_internal_process()`
- **变量名**：使用小写下划线命名法，如`vertex_count`、`material_index`
- **常量**：使用全大写下划线格式，如`MAX_VERTICES`、`DEFAULT_COLOR`

### 命名最佳实践

- 名称应具有描述性，清楚表明用途
- 避免使用缩写，除非是广泛接受的（如UI、ID）
- 保持与Blender内置命名风格的一致性
- 在命名中避免使用Blender关键字，防止冲突

```python
# 命名示例
class OBJECT_OT_add_special_cube(bpy.types.Operator):
    bl_idname = "object.add_special_cube"
    bl_label = "添加特殊立方体"
    
    cube_size: bpy.props.FloatProperty(name="立方体大小")
    material_type: bpy.props.EnumProperty(
        items=[
            ('BASIC', "基础", "基础材质"),
            ('ADVANCED', "高级", "高级材质")
        ]
    )
    
    def execute(self, context):
        # 变量命名示例
        vertex_count = 8
        DEFAULT_COLOR = (1.0, 1.0, 1.0)
        return {'FINISHED'}
```

## 文件夹组织规范

### 插件根目录组织

- 保持根目录简洁，只包含必要的Python模块和资源目录
- 使用标准化的目录结构，便于理解和维护
- 模块化组织代码，按功能拆分为多个Python文件

### 标准目录结构

```
my_addon/
│
├── __init__.py         # 插件入口，包含bl_info和注册函数
├── operators/          # 操作符实现目录
│   ├── __init__.py     # 导入所有操作符
│   ├── create.py       # 创建类操作符
│   └── modify.py       # 修改类操作符
│
├── ui/                 # 界面相关目录
│   ├── __init__.py     # 导入所有UI元素
│   ├── panels.py       # 面板定义
│   └── menus.py        # 菜单定义
│
├── properties/         # 属性定义目录
│   ├── __init__.py
│   └── properties.py   # 属性和属性组定义
│
├── core/               # 核心功能目录
│   ├── __init__.py
│   ├── utils.py        # 实用工具函数
│   └── algorithms.py   # 核心算法实现
│
├── presets/            # 预设目录
│   └── default.py      # 默认预设
│
├── resources/          # 资源目录
│   ├── icons/          # 图标资源
│   ├── templates/      # 模板文件
│   └── config.json     # 配置文件
│
└── lib/                # 第三方库目录
    └── external_module/  # 外部依赖模块
```

### 资源文件组织

- **图标资源**：放置在`resources/icons/`目录，使用统一的格式（建议PNG）
- **预设文件**：放置在`presets/`目录，便于用户选择不同配置
- **配置文件**：如需持久化配置，使用JSON或YAML格式存储在`resources/`目录
- **临时文件**：在操作系统临时目录创建和管理，不要放在插件目录内

### 第三方库处理

- **内置库**：直接放置在`lib/`目录下，确保相对导入路径正确
- **纯Python库**：可直接包含在插件包中
- **带编译组件的库**：需要为不同操作系统提供预编译版本，或在安装时编译
- **常用库处理**：

```python
# 第三方库导入示例
try:
    # 先尝试从标准路径导入
    import numpy as np
except ImportError:
    # 如果失败，从插件lib目录导入
    import os
    import sys
    lib_path = os.path.join(os.path.dirname(__file__), "lib")
    if lib_path not in sys.path:
        sys.path.append(lib_path)
    import numpy as np
```

### 多文件模块组织

- 使用`__init__.py`聚合子模块中的类和函数
- 通过命名空间组织代码，避免导入时的命名冲突
- 示例导入结构：

```python
# __init__.py 示例
from .operators.create import OBJECT_OT_add_special_cube
from .operators.modify import OBJECT_OT_modify_special_cube
from .ui.panels import VIEW3D_PT_special_tools
from .properties.properties import SpecialProperties

# 统一注册所有类
classes = (
    OBJECT_OT_add_special_cube,
    OBJECT_OT_modify_special_cube,
    VIEW3D_PT_special_tools,
    SpecialProperties
)

def register():
    from bpy.utils import register_class
    for cls in classes:
        register_class(cls)

def unregister():
    from bpy.utils import unregister_class
    for cls in reversed(classes):
        unregister_class(cls)
```

### 配置文件使用

- 使用配置文件存储默认设置和用户偏好
- 支持配置文件的保存和加载
- 示例配置管理：

```python
import os
import json

def get_config_path():
    """获取配置文件路径"""
    addon_dir = os.path.dirname(os.path.realpath(__file__))
    return os.path.join(addon_dir, "resources", "config.json")

def load_config():
    """加载配置"""
    config_path = get_config_path()
    if os.path.exists(config_path):
        with open(config_path, 'r') as f:
            return json.load(f)
    return {} # 默认配置

def save_config(config):
    """保存配置"""
    config_path = get_config_path()
    os.makedirs(os.path.dirname(config_path), exist_ok=True)
    with open(config_path, 'w') as f:
        json.dump(config, f, indent=4)
```

## 关键组件

### 操作符（Operator）

操作符是插件的功能执行单元，需定义以下方法：

- **poll()**：判断操作符是否可用的静态方法
- **execute()**：执行操作的主要方法
- **invoke()**：在执行前调用，通常用于设置参数或显示对话框
- **modal()**：用于实现模态操作（连续交互）

```python
class MyOperator(bpy.types.Operator):
    bl_idname = "object.my_operator"
    bl_label = "操作名称"
    bl_options = {'REGISTER', 'UNDO'}
    
    # 属性定义
    my_prop: bpy.props.FloatProperty(name="参数名称")
    
    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    
    def execute(self, context):
        # 功能实现代码
        return {'FINISHED'}
```

### 面板（Panel）

面板用于提供用户界面，需定义以下内容：

- **bl_space_type**：面板所在空间（如VIEW_3D、PROPERTIES）
- **bl_region_type**：面板所在区域（如UI、TOOLS）
- **bl_category**：面板所在选项卡名称
- **draw()**：绘制界面的方法

```python
class MyPanel(bpy.types.Panel):
    bl_idname = "VIEW3D_PT_my_panel"
    bl_label = "我的面板"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "我的插件"
    
    def draw(self, context):
        layout = self.layout
        layout.operator("object.my_operator")
```

### 属性（Property）

使用`bpy.props`模块定义属性，常见类型：

- **FloatProperty**：浮点数属性
- **IntProperty**：整数属性
- **BoolProperty**：布尔属性
- **StringProperty**：字符串属性
- **EnumProperty**：枚举选项属性
- **PointerProperty**：指向其他数据的指针
- **CollectionProperty**：集合属性

### 属性组（PropertyGroup）

用于组织复杂属性结构：

```python
class MySettings(bpy.types.PropertyGroup):
    size: bpy.props.FloatProperty(
        name="尺寸",
        default=1.0,
        min=0.1,
        max=10.0
    )
    
    mode: bpy.props.EnumProperty(
        name="模式",
        items=[
            ('OPTION1', "选项1", "第一个选项描述"),
            ('OPTION2', "选项2", "第二个选项描述"),
        ],
        default='OPTION1'
    )
```

## API使用规范

### 核心API模块

- **bpy**：主要Blender Python API
- **bmesh**：网格编辑API
- **mathutils**：数学工具（向量、矩阵等）
- **bgl/gpu**：OpenGL相关功能
- **addon_utils**：插件工具函数

### 数据访问规范

- 使用`context`获取当前上下文数据
- 使用`bpy.data`访问场景中的数据块
- 使用`object.data`访问物体的网格数据
- 推荐使用安全访问方式，确保对象存在再访问其属性

### 错误处理

- 使用`try-except`块处理可能的错误
- 使用`self.report()`向用户报告错误
- 记录详细错误信息到控制台供调试

```python
try:
    # 操作代码
except Exception as e:
    self.report({'ERROR'}, f"发生错误: {str(e)}")
    return {'CANCELLED'}
```

## 界面设计

### 布局系统

- 使用`layout.row()`创建行布局
- 使用`layout.column()`创建列布局
- 使用`layout.box()`创建框布局
- 使用`layout.split()`分割布局

### UI最佳实践

- 使用分组和标签组织相关控件
- 采用一致的对齐和间距
- 提供清晰的控件提示信息
- 避免过于拥挤的界面设计
- 合理使用图标增强可视性

```python
def draw(self, context):
    layout = self.layout
    settings = context.scene.my_addon_settings
    
    # 主要设置
    box = layout.box()
    box.label(text="基本设置")
    
    row = box.row()
    row.prop(settings, "size")
    
    # 高级设置
    box = layout.box()
    box.label(text="高级选项")
    box.prop(settings, "mode")
    
    # 操作按钮
    layout.separator()
    layout.operator("object.my_operator")
```

## UI设计可视化

在Blender插件开发过程中，AI和用户之间关于UI界面设计的沟通至关重要。本节提供了一系列方法，帮助AI以可视化方式表达UI设计方案，提高沟通效率。

### ASCII界面布局

使用ASCII字符创建UI布局示意图，帮助用户快速理解界面结构：

```
+---------------------------+  <- 面板标题
| ● 网格设置                 |  <- 折叠组标题
|   [数量: 10      ]        |  <- 滑块属性
|   [大小: 2.0     ]        |  
|   (x) 正方形 ( ) 圆形     |  <- 单选按钮
|                           |
| ● 材质选项                 |  
|   [材质: ▼ 金属  ]        |  <- 下拉菜单
|   [√] 使用法线贴图        |  <- 复选框
|                           |
| [  生成网格  ]             |  <- 按钮
+---------------------------+
```

### 表格式布局描述

使用表格明确地描述UI组件层次和属性：

| 组件类型 | 标签 | 属性/值 | 父组件 | 说明 |
|---------|-----|--------|-------|-----|
| Panel | 我的工具 | bl_category="工具" | - | 主面板 |
| Box | 基本设置 | - | Panel | 设置分组 |
| Prop | 尺寸 | FloatProperty, default=1.0 | Box | 控制尺寸参数 |
| Operator | 应用 | operator="object.my_operator" | Panel | 执行主要功能 |

### 组件符号系统

使用简单的符号系统表示各种UI组件：

- `[ ]` - 文本输入框
- `[▼]` - 下拉菜单
- `(•)` - 单选按钮（选中）
- `( )` - 单选按钮（未选中）
- `[✓]` - 复选框（选中）
- `[ ]` - 复选框（未选中）
- `◉` - 颜色选择器
- `━━` - 滑块
- `[BTN]` - 按钮

### 界面层次描述

明确描述界面组件的嵌套关系：

```
面板: "模型工具"
├── 折叠组: "创建选项"
│   ├── 属性: "尺寸" (Float)
│   ├── 属性: "细分" (Int)
│   └── 操作符: "创建"
├── 折叠组: "编辑选项"
│   ├── 属性: "强度" (Float)
│   └── 操作符: "应用"
└── 操作符: "重置"
```

### 布局尺寸和对齐可视化

示意布局的尺寸和对齐方式：

```
[属性标签     ][     输入框     ]  <- 等分行
[长标签文本   ][小输入]            <- 比例分配
[     全宽按钮操作符     ]         <- 全宽组件
```

### 交互流程图

表示UI组件间的交互关系和数据流：

```
[尺寸输入] ──┐
[细分输入] ──┼──> [创建按钮] ──> [生成结果]
[类型选择] ──┘
```

### UI状态变化示意

描述UI组件在不同状态下的变化：

```
初始状态:
[✓] 高级选项 
    [密度: 10]
    [平滑处理]
    
切换后:
[ ] 高级选项
    (高级选项被隐藏)
```

### 颜色和图标表示

使用文本描述颜色和图标：

```
[📊 分析](浅蓝色按钮)
[🔧 设置](灰色按钮)
[❗ 警告](红色文本)
```

### 实际UI截图引用

当Blender中存在类似UI组件时，可以引用其官方名称：

"此界面使用与Blender材质编辑器中类似的节点连接系统，用户可以连接不同的属性节点。"

### 渐进式设计沟通

提供UI设计的迭代方案，从简单到复杂：

1. **基础版本**：仅包含核心功能的最简界面
2. **标准版本**：增加分组和常用选项
3. **高级版本**：包含所有功能和自定义选项

### 模拟屏幕截图

使用代码块和ASCII艺术创建模拟的屏幕截图：

```
+------ Blender 视图 ------+
|                         |
|    [我的插件]           |
|    +----------------+   |
|    | ● 基本设置     |   |
|    | [X: 0.0     ]  |   |
|    | [Y: 0.0     ]  |   |
|    | [Z: 0.0     ]  |   |
|    |                |   |
|    | [  应用设置  ] |   |
|    +----------------+   |
|                         |
+-------------------------+
```

### 用户反馈迭代设计

在设计沟通中设置明确的反馈点：

```
设计反馈点:
1. 您更喜欢选项分组在一个面板中还是拆分为多个标签页?
2. 控制点参数应该使用滑块还是数值输入框?
3. 以下哪种布局更适合您的工作流程: A, B, C?
```

### 组合应用示例

以下是综合运用多种可视化技术的复杂界面示例：

```
# 曲线生成器插件界面

+---------------------------+
| ● 曲线参数                 |
|   类型: [▼ 贝塞尔曲线]     |
|   点数: [━━━━●━━━] 8      |
|   闭合: [✓]               |
|                           |
| ● 细分设置                 |
|   分辨率: [━━●━━━━] 24    |
|   平滑度: [━━━━━●━] 0.7   |
|                           |
| ● 显示选项                 |
|   [✓] 显示控制点          |
|   [✓] 显示法线            |
|   [ ] 显示曲率            |
|                           |
| [   预览   ] [   生成   ]  |
+---------------------------+

交互流程:
1. 用户选择曲线类型
2. 调整点数和其他参数
3. 点击预览查看结果
4. 满意后点击生成创建实际对象

状态依赖:
- "显示法线"选项仅在"显示控制点"选中时可用
- 点数超过12时会显示性能警告
```

### 使用可视化UI的最佳实践

- **循序渐进**：先展示UI的整体结构，再展示细节
- **一致性**：使用统一的符号和表示方法
- **组件对应**：明确指出UI组件与代码中类和属性的对应关系
- **考虑工作流**：在设计中体现用户的使用流程
- **保持简洁**：使用简单直观的表示方法，避免过度复杂的可视化

### 布局代码与可视化对应

展示UI可视化和实际代码的对应关系：

```
# 可视化界面
+-----------------------+
| [尺寸: 1.0     ]      |
| [  应用  ]            |
+-----------------------+

# 对应代码
def draw(self, context):
    layout = self.layout
    layout.prop(context.scene.my_tool, "size")
    layout.operator("object.apply_size")
```

## 性能优化

### 优化技巧

- **减少更新**：使用`context.window_manager.modal_handler_add()`实现模态操作
- **缓存数据**：避免重复计算相同的值
- **延迟加载**：仅在需要时导入重量级模块
- **内存管理**：及时释放不再需要的大型数据结构
- **批量处理**：对多个物体的操作尽量批量进行

### 耗时操作处理

- 使用进度条提示长时间操作
- 考虑实现取消功能
- 将复杂计算放在后台线程（注意Blender的线程安全性）

```python
def process_with_progress(self, context, items):
    wm = context.window_manager
    total = len(items)
    
    wm.progress_begin(0, total)
    
    for i, item in enumerate(items):
        # 处理单个项目
        process_item(item)
        
        # 更新进度
        wm.progress_update(i)
        
        # 检查是否取消
        if self.is_cancelled:
            break
    
    wm.progress_end()
```

## 测试与调试

### 测试方法

- **手动测试**：在不同场景和环境下测试功能
- **自动测试**：编写单元测试检查核心功能
- **极限测试**：测试边界条件和极端参数

### 调试技巧

- 使用`print()`输出调试信息
- 使用Blender的Python控制台检查变量
- 设置断点进行交互式调试
- 使用`import traceback; traceback.print_exc()`打印详细错误信息

### 常见错误排查

- 检查类型错误：确保变量类型正确
- 检查空值错误：确保对象存在后再访问
- 检查上下文错误：确保在正确的上下文中执行操作
- 检查注册/注销错误：确保所有类都已正确注册

## 文档规范

### 代码注释

- 使用文档字符串（docstring）描述类和函数的功能
- 注释复杂算法的实现原理
- 标记特殊处理和临时解决方案

```python
def complex_function(param1, param2):
    """
    函数功能描述
    
    Args:
        param1: 参数1描述
        param2: 参数2描述
        
    Returns:
        返回值描述
        
    Raises:
        可能的异常
    """
    # 实现代码
```

### 用户文档

- 提供插件安装指南
- 提供主要功能的使用说明
- 提供参数和选项的解释
- 推荐包含示例和教程链接

## 代码移植与重构

### 从其他软件移植代码

将其他3D软件（如Maya、3ds Max、Houdini等）的功能移植到Blender时，需遵循以下步骤和规范：

#### 概念映射

- **理解概念差异**：不同软件使用不同术语表示相似概念
  - 例如：Maya的"Mesh" ≈ Blender的"Mesh"，但内部结构不同
  - 3ds Max的"Modifier" ≈ Blender的"Modifier"
  - Houdini的"Node" ≈ Blender的"Node"（在几何节点或材质节点中）

- **常见概念对照表**：

| 其他软件      | Blender对应         | 注意事项                           |
|-------------|--------------------|----------------------------------|
| Maya Scene  | bpy.context.scene   | Blender中场景管理方式不同           |
| Max Object  | bpy.data.objects    | 对象系统可能有不同层级结构           |
| Vertices    | mesh.vertices       | 顶点索引和访问方式不同               |
| Material    | bpy.data.materials  | 材质系统架构有根本差异               |
| Transform   | object.matrix_world | 坐标系和旋转表示可能不同             |

#### 算法翻译策略

1. **功能分析**：首先理解原始代码实现的功能本质
2. **流程图绘制**：绘制算法流程图，不依赖于特定软件的API
3. **API替换**：用Blender API替换原始软件的API调用
4. **数据结构转换**：处理不同软件间的数据结构差异
5. **坐标系调整**：处理可能的坐标系差异（如Z轴向上vs. Y轴向上）

```python
# Maya代码示例
# selected = cmds.ls(selection=True)
# cmds.polySmooth(selected, divisions=2)

# 转换为Blender代码
import bpy

# 获取选中物体
selected = bpy.context.selected_objects
mesh_objects = [obj for obj in selected if obj.type == 'MESH']

# 应用平滑修改器
for obj in mesh_objects:
    modifier = obj.modifiers.new(name="Smooth", type='SMOOTH')
    modifier.iterations = 2
```

#### 常见挑战与解决方案

- **插件系统差异**：理解Blender的插件架构与其他软件不同
- **性能考量**：重写算法时优化性能，利用Blender特有的优化方式
- **用户界面转换**：重新设计符合Blender界面风格的UI
- **工作流程调整**：调整功能以适应Blender的工作流程

### 从其他Blender插件移植功能

从现有Blender插件中提取和移植功能时的最佳实践：

#### 代码分析与重用

- **版权检查**：确认原始插件许可是否允许代码重用
- **依赖识别**：识别插件的所有必要依赖和导入
- **提取核心算法**：提取所需功能的关键代码，而非直接复制整个插件
- **适当引用**：在注释和文档中引用原始代码来源

```python
# 原始代码：
class ORIGINAL_OT_function(bpy.types.Operator):
    # ... 复杂实现 ...
    
    def execute(self, context):
        # 这是我们需要的核心算法
        result = self.complex_algorithm(context.object)
        return {'FINISHED'}
        
    def complex_algorithm(self, obj):
        # 算法实现
        return result

# 移植后的代码：
"""
此函数基于Original Add-on (https://example.org)，
根据其MIT许可进行修改和使用。
"""
def adapted_algorithm(obj):
    # 调整后的算法实现
    return result

class NEW_OT_function(bpy.types.Operator):
    # ... 新的实现 ...
    
    def execute(self, context):
        result = adapted_algorithm(context.object)
        return {'FINISHED'}
```

#### 功能整合方法

1. **模块化重构**：将原代码重构为模块化组件再整合
2. **接口统一**：统一不同来源代码的接口和命名规范
3. **移除冗余**：合并重复功能，移除不必要部分
4. **一致性检查**：确保移植的功能与插件其他部分风格一致

#### API版本兼容性

- **API变更处理**：处理不同Blender版本间的API变化
- **版本检测**：添加版本检测代码处理不同API版本

```python
import bpy

def perform_operation(obj):
    # 处理API版本差异
    if bpy.app.version >= (2, 80, 0):
        # 2.80及以上版本的代码
        obj.select_set(True)
    else:
        # 2.79及以下版本的代码
        obj.select = True
```

### 单文件向多文件结构转换

将单个.py脚本文件拆分为规范的多文件结构：

#### 重构步骤

1. **代码分析**：识别代码中的逻辑模块和功能组
2. **目录创建**：建立标准目录结构（如前面"文件夹组织规范"章节所示）
3. **代码分离**：按功能将代码拆分到不同文件
4. **模块链接**：建立正确的导入关系

#### 代码拆分原则

- **按功能分离**：将不同功能的代码分离到专用文件
- **保持内聚性**：每个文件应包含紧密相关的功能
- **最小化依赖**：减少文件间的循环依赖
- **逻辑分组**：将相关组件分组到同一子目录

#### 单文件到多文件转换示例

原始单文件结构：
```python
# my_addon.py
bl_info = {...}

import bpy

# 属性定义
class MyProperties(bpy.types.PropertyGroup):
    # ...属性定义...

# 操作符定义
class MY_OT_operator1(bpy.types.Operator):
    # ...操作符实现...

class MY_OT_operator2(bpy.types.Operator):
    # ...另一个操作符...

# 面板定义
class MY_PT_panel(bpy.types.Panel):
    # ...面板实现...

# 注册函数
def register():
    # ...注册代码...

def unregister():
    # ...注销代码...

if __name__ == "__main__":
    register()
```

转换为多文件结构：

1. 创建基本目录结构：
```
my_addon/
├── __init__.py
├── operators.py
├── panels.py
└── properties.py
```

2. 将代码分配到相应文件：

`__init__.py`:
```python
bl_info = {...}

import bpy
from . import operators
from . import panels
from . import properties

def register():
    bpy.utils.register_class(properties.MyProperties)
    operators.register()
    panels.register()
    bpy.types.Scene.my_addon = bpy.props.PointerProperty(type=properties.MyProperties)

def unregister():
    panels.unregister()
    operators.unregister()
    bpy.utils.unregister_class(properties.MyProperties)
    del bpy.types.Scene.my_addon

if __name__ == "__main__":
    register()
```

`properties.py`:
```python
import bpy

class MyProperties(bpy.types.PropertyGroup):
    # ...属性定义...
```

`operators.py`:
```python
import bpy

class MY_OT_operator1(bpy.types.Operator):
    # ...操作符实现...

class MY_OT_operator2(bpy.types.Operator):
    # ...另一个操作符...

classes = (
    MY_OT_operator1,
    MY_OT_operator2,
)

def register():
    from bpy.utils import register_class
    for cls in classes:
        register_class(cls)

def unregister():
    from bpy.utils import unregister_class
    for cls in reversed(classes):
        unregister_class(cls)
```

`panels.py`:
```python
import bpy

class MY_PT_panel(bpy.types.Panel):
    # ...面板实现...

classes = (
    MY_PT_panel,
)

def register():
    from bpy.utils import register_class
    for cls in classes:
        register_class(cls)

def unregister():
    from bpy.utils import unregister_class
    for cls in reversed(classes):
        unregister_class(cls)
```

### 多文件向单文件结构合并

将复杂的多文件插件简化为单个脚本文件：

#### 合并策略

1. **代码依赖分析**：分析并理解模块间的依赖关系
2. **合并顺序确定**：确定正确的代码合并顺序
3. **冗余代码移除**：移除重复和未使用的代码
4. **注释增强**：通过注释和章节标记保持代码组织清晰

#### 合并步骤示例

将标准多文件结构合并为单文件：

```python
# my_addon.py

# === 插件信息 ===
bl_info = {...}

import bpy

# === 工具函数 ===
# (从utils.py合并)
def util_function1():
    # ...实现...
    pass

def util_function2():
    # ...实现...
    pass

# === 属性定义 ===
# (从properties.py合并)
class MyProperties(bpy.types.PropertyGroup):
    # ...属性定义...

# === 操作符 ===
# (从operators.py合并)
class MY_OT_operator1(bpy.types.Operator):
    # ...操作符实现...

class MY_OT_operator2(bpy.types.Operator):
    # ...另一个操作符...

# === 面板 ===
# (从panels.py合并)
class MY_PT_panel(bpy.types.Panel):
    # ...面板实现...

# === 注册/注销 ===
classes = (
    MyProperties,
    MY_OT_operator1,
    MY_OT_operator2,
    MY_PT_panel,
)

def register():
    from bpy.utils import register_class
    for cls in classes:
        register_class(cls)
    bpy.types.Scene.my_addon = bpy.props.PointerProperty(type=MyProperties)

def unregister():
    from bpy.utils import unregister_class
    for cls in reversed(classes):
        unregister_class(cls)
    del bpy.types.Scene.my_addon

if __name__ == "__main__":
    register()
```

#### 保持可维护性

- **使用注释分隔功能区块**：清晰标记不同功能区域
- **保持代码顺序逻辑**：按依赖关系和逻辑顺序排列代码
- **移除内部导入**：调整所有内部模块导入为直接引用
- **适当简化**：移除过度模块化造成的复杂性

### 重构与移植的最佳实践

- **循序渐进**：一次处理一个组件，避免大规模重写
- **频繁测试**：每完成一部分就进行测试，确保功能正常
- **版本控制**：使用Git等工具，便于跟踪变更和回滚
- **保持兼容性**：考虑向后兼容性，特别是对现有数据的处理
- **代码审查**：重构后进行代码审查，确保质量和一致性
- **文档更新**：随代码变更更新相关文档

## 发布与分发

### 打包规范

- 移除测试代码和调试语句
- 优化导入语句和代码结构
- 确保所有依赖都已包含或在文档中说明
- 创建适当的__init__.py导入所有必要模块

### 版本控制

- 遵循语义化版本规范（X.Y.Z）
- 记录版本变更日志
- 在bl_info中更新版本号

### 发布渠道

- Blender Market（商业插件）
- Gumroad（商业插件）
- GitHub（开源插件）
- Blender插件目录（官方认可的插件）

## 常见问题解决

### 常见错误与解决方案

- **插件加载失败**：检查bl_info格式、Python语法错误
- **注册失败**：检查类定义和继承关系、类型注册顺序
- **功能无效**：检查上下文条件、操作符poll方法
- **UI不显示**：检查面板定义和空间类型配置

### 兼容性问题处理

- 使用`bpy.app.version`检查Blender版本
- 针对不同版本提供兼容代码分支
- 记录已测试的Blender版本范围

```python
if bpy.app.version >= (2, 93, 0):
    # 2.93及以上版本的代码
else:
    # 兼容旧版本的代码
```

## 示例模板

### 基础插件模板

```python
bl_info = {
    "name": "示例插件",
    "author": "AI助手",
    "version": (1, 0, 0),
    "blender": (2, 93, 0),
    "location": "视图3D > 侧边栏 > 示例",
    "description": "示例Blender插件模板",
    "category": "Object",
}

import bpy

# 属性定义
class MyAddonProperties(bpy.types.PropertyGroup):
    my_float: bpy.props.FloatProperty(
        name="参数",
        description="参数描述",
        default=1.0,
        min=0.0,
        max=10.0
    )

# 操作符
class OBJECT_OT_my_operator(bpy.types.Operator):
    bl_idname = "object.my_operator"
    bl_label = "示例操作"
    bl_options = {'REGISTER', 'UNDO'}
    
    @classmethod
    def poll(cls, context):
        return context.active_object is not None
    
    def execute(self, context):
        obj = context.active_object
        settings = context.scene.my_addon_settings
        
        # 示例操作：缩放物体
        obj.scale *= settings.my_float
        
        self.report({'INFO'}, f"已将物体缩放 {settings.my_float} 倍")
        return {'FINISHED'}

# 面板
class VIEW3D_PT_my_panel(bpy.types.Panel):
    bl_label = "示例面板"
    bl_idname = "VIEW3D_PT_my_panel"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "示例"
    
    def draw(self, context):
        layout = self.layout
        settings = context.scene.my_addon_settings
        
        layout.prop(settings, "my_float")
        layout.operator("object.my_operator")

# 类注册列表
classes = (
    MyAddonProperties,
    OBJECT_OT_my_operator,
    VIEW3D_PT_my_panel,
)

def register():
    for cls in classes:
        bpy.utils.register_class(cls)
    bpy.types.Scene.my_addon_settings = bpy.props.PointerProperty(type=MyAddonProperties)

def unregister():
    for cls in reversed(classes):
        bpy.utils.unregister_class(cls)
    del bpy.types.Scene.my_addon_settings

if __name__ == "__main__":
    register()
```

---

本规范指南旨在帮助AI生成高质量、标准化的Blender插件代码，遵循最佳实践和开发规范。使用本指南可以确保生成的插件具有良好的结构、性能和可维护性，同时符合Blender插件生态系统的要求和期望。 