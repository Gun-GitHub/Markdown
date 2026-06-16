# ComfyUI 自定义节点开发指南

> 本文档将带您从零开始，掌握 ComfyUI 自定义节点的创建方法。
> 最后更新：2026-05-13

---

## 目录

- [1. 前置知识](#1-前置知识)
- [2. 项目结构](#2-项目结构)
- [3. 最简节点示例](#3-最简节点示例)
- [4. 节点类详解](#4-节点类详解)
- [5. 输入输出类型](#5-输入输出类型)
- [6. 常见节点类型](#6-常见节点类型)
- [7. 高级特性](#7-高级特性)
- [8. 调试与排错](#8-调试与排错)
- [9. 发布与分发](#9-发布与分发)

---

## 1. 前置知识

### 1.1 技术栈要求

| 项目 | 要求 |
|------|------|
| Python | 3.10+ |
| PyTorch | 2.0+ |
| ComfyUI | 最新稳定版 |

### 1.2 ComfyUI 节点工作原理

ComfyUI 采用 **数据流图 (Dataflow Graph)** 架构：

```
[输入节点] → [处理节点] → [处理节点] → [输出节点]
     ↓            ↓            ↓            ↓
   数据       转换数据     进一步处理     结果输出
```

每个节点是一个独立的 Python 类，包含：
- **输入定义**：接受哪些类型的数据
- **执行逻辑**：`execute()` 或 `forward()` 方法
- **输出定义**：返回什么类型的数据

---

## 2. 项目结构

### 2.1 标准目录结构

```
ComfyUI/custom_nodes/
└── my_custom_nodes/
    ├── __init__.py          # 节点注册入口
    ├── nodes.py             # 节点类定义
    ├── pyproject.toml       # 项目元数据（可选）
    ├── requirements.txt     # 依赖包（可选）
    ├── README.md            # 说明文档
    └── example_workflows/   # 示例工作流（可选）
        └── example.json
```

### 2.2 快速创建项目骨架

```bash
cd ~/ComfyUI/custom_nodes
git clone <your-node-repo> my_custom_nodes
cd my_custom_nodes
pip install -r requirements.txt
```

---

## 3. 最简节点示例

### 3.1 节点类定义 (`nodes.py`)

```python
"""
ComfyUI 自定义节点示例 - 最简版本
"""

import torch


class SimpleTextConcat:
    """
    文本拼接节点
    将两个输入文本拼接为一个输出文本
    """

    # 类属性：定义节点显示名称
    FUNCTION_NAME = "Simple Text Concat"

    def __init__(self):
        pass

    @classmethod
    def INPUT_TYPES(cls):
        """
        定义节点的输入参数
        返回字典，键为参数名，值为参数配置
        """
        return {
            "required": {
                "text_a": ("STRING", {
                    "default": "Hello",
                    "multiline": False,
                    "tooltip": "第一个文本输入"
                }),
                "text_b": ("STRING", {
                    "default": "World",
                    "multiline": False,
                    "tooltip": "第二个文本输入"
                }),
                "separator": ("STRING", {
                    "default": " ",
                    "multiline": False,
                    "tooltip": "分隔符"
                }),
            },
            "optional": {
                "upper_case": ("BOOLEAN", {
                    "default": False,
                    "tooltip": "是否转为大写"
                }),
            }
        }

    RETURN_TYPES = ("STRING",)
    RETURN_NAMES = ("concatenated_text",)
    FUNCTION = "concatenate"
    CATEGORY = "utils/text"
    DESCRIPTION = "将两个文本拼接在一起"

    def concatenate(self, text_a, text_b, separator, upper_case=False):
        """
        节点执行逻辑
        """
        result = f"{text_a}{separator}{text_b}"
        if upper_case:
            result = result.upper()
        return (result,)


# 节点注册字典
NODE_CLASS_MAPPINGS = {
    "SimpleTextConcat": SimpleTextConcat,
}

# 节点显示名称映射
NODE_DISPLAY_NAME_MAPPINGS = {
    "SimpleTextConcat": "文本拼接",
}
```

### 3.2 注册入口 (`__init__.py`)

```python
"""
ComfyUI 自定义节点包入口
"""

# 从 nodes.py 导入注册字典
from .nodes import NODE_CLASS_MAPPINGS, NODE_DISPLAY_NAME_MAPPINGS

# 导出给 ComfyUI 使用
__all__ = ['NODE_CLASS_MAPPINGS', 'NODE_DISPLAY_NAME_MAPPINGS']

# 包信息
WEB_DIRECTORY = "./web"  # 如果有前端资源
VERSION = "1.0.0"
AUTHOR = "Your Name"

print(f"[CustomNode] {__name__} v{VERSION} loaded")
```

---

## 4. 节点类详解

### 4.1 核心属性与方法

```python
class MyNode:
    """
    节点类模板
    """

    # ===== 必需属性 =====

    RETURN_TYPES = ()          # 元组，定义返回类型
    FUNCTION = "execute"       # 字符串，指定执行方法名

    # ===== 推荐属性 =====

    CATEGORY = "custom"        # 节点在侧边栏的分类
    DESCRIPTION = ""           # 节点描述（悬停提示）
    OUTPUT_IS_LIST = (False,)  # 每个输出是否为列表

    # ===== 必需方法 =====

    @classmethod
    def INPUT_TYPES(cls):
        """定义输入参数"""
        return {
            "required": {},    # 必需参数
            "optional": {},    # 可选参数
            "hidden": {},      # 隐藏参数（内部使用）
        }

    def execute(self, **kwargs):
        """
        执行逻辑
        返回值必须是元组，长度与 RETURN_TYPES 一致
        """
        pass
```

### 4.2 输入参数配置详解

```python
@classmethod
def INPUT_TYPES(cls):
    return {
        "required": {
            # 字符串输入
            "text_input": ("STRING", {
                "default": "",
                "multiline": True,        # 是否多行文本框
                "tooltip": "说明文字",
                "dynamicPrompts": True,   # 是否支持动态提示词
            }),

            # 整数输入
            "width": ("INT", {
                "default": 512,
                "min": 64,
                "max": 8192,
                "step": 64,
                "display": "number",      # "number" 或 "slider"
                "tooltip": "图像宽度",
            }),

            # 浮点数输入
            "strength": ("FLOAT", {
                "default": 0.8,
                "min": 0.0,
                "max": 1.0,
                "step": 0.01,
                "round": 0.01,
                "display": "slider",
                "tooltip": "强度值",
            }),

            # 布尔值输入
            "enable": ("BOOLEAN", {
                "default": True,
                "tooltip": "是否启用",
            }),

            # 下拉选择
            "mode": (["auto", "manual", "advanced"], {
                "default": "auto",
                "tooltip": "运行模式",
            }),

            # 组合框（动态选项）
            "model": (COMBO_MODEL_LIST, {
                "image_upload": False,
                "tooltip": "选择模型",
            }),

            # 图像输入
            "image": ("IMAGE", {}),

            # 条件输入
            "positive": ("CONDITIONING", {}),
            "negative": ("CONDITIONING", {}),

            # 潜空间输入
            "latent": ("LATENT", {}),

            # 模型输入
            "model": ("MODEL", {}),

            # 钳位模式输入
            "clips": ("CLIP", {}),
        },
        "optional": {
            # 可选参数定义方式相同
            "seed": ("INT", {"default": 0, "min": 0, "max": 0xFFFFFFFFFFFFFFFF}),
        },
        "hidden": {
            # 隐藏参数，通常用于内部传递
            "unique_id": "UNIQUE_ID",
            "extra_pnginfo": "EXTRA_PNGINFO",
        }
    }
```

### 4.3 动态选项（Combo）的实现

```python
# 方式一：静态列表
MODEL_LIST = ["model_a.safetensors", "model_b.safetensors", "model_c.safetensors"]

# 方式二：动态获取（在 INPUT_TYPES 中调用类方法）
@classmethod
def INPUT_TYPES(cls):
    models = cls.get_available_models()
    return {
        "required": {
            "model": (models, {"default": models[0] if models else ""}),
        }
    }

@classmethod
def get_available_models(cls):
    """动态获取可用模型列表"""
    import os
    models_dir = os.path.join(os.path.dirname(__file__), "..", "..", "models", "checkpoints")
    if os.path.exists(models_dir):
        return sorted([
            f for f in os.listdir(models_dir)
            if f.endswith(('.safetensors', '.ckpt'))
        ])
    return []
```

---

## 5. 输入输出类型

### 5.1 内置数据类型速查表

| 类型名 | 说明 | 典型用途 |
|--------|------|----------|
| `STRING` | 文本字符串 | 提示词、路径、参数 |
| `INT` | 整数 | 尺寸、数量、种子 |
| `FLOAT` | 浮点数 | 比例、强度、阈值 |
| `BOOLEAN` | 布尔值 | 开关、启用/禁用 |
| `IMAGE` | 图像张量 | 图像数据 `[B, H, W, C]` |
| `LATENT` | 潜空间数据 | 扩散模型中间表示 |
| `CONDITIONING` | 条件编码 | 正/负提示词编码 |
| `MODEL` | 模型对象 | 检查点模型 |
| `CLIP` | CLIP 编码器 | 文本编码器 |
| `VAE` | VAE 编解码器 | 潜空间编解码 |
| `CONTROL_NET` | 控制网 | 姿态/边缘控制 |
| `MASK` | 掩码 | 图像遮罩 |
| `GLIGEN` | GLIGEN 对象 | 位置文本控制 |

### 5.2 自定义数据类型

```python
# 定义自定义类型
CUSTOM_DATA_TYPE = "MyCustomData"

class MyNode:
    RETURN_TYPES = (CUSTOM_DATA_TYPE,)
    FUNCTION = "execute"
    CATEGORY = "custom"

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "input_data": (CUSTOM_DATA_TYPE, {}),
            }
        }

    def execute(self, input_data):
        # 处理自定义数据
        return (input_data,)
```

### 5.3 图像数据格式

```python
# ComfyUI 图像张量格式：[Batch, Height, Width, Channels]
# 值范围：0.0 ~ 1.0 (float32)
# 通道顺序：RGB

# 创建示例图像
def create_test_image(width=512, height=512):
    # 创建随机图像
    img = torch.rand(1, height, width, 3)
    return (img,)

# 从 PIL 转换
from PIL import Image
import numpy as np

def pil_to_comfyui_image(pil_img):
    img_array = np.array(pil_img, dtype=np.float32) / 255.0
    img_tensor = torch.from_numpy(img_array)[None, :]  # 添加 batch 维度
    if img_tensor.shape[3] == 4:  # RGBA -> RGB
        img_tensor = img_tensor[:, :, :, :3]
    return (img_tensor,)

# 从 ComfyUI 图像转换到 PIL
def comfyui_image_to_pil(image_tensor):
    img_array = (image_tensor[0].cpu().numpy() * 255).astype(np.uint8)
    return Image.fromarray(img_array)
```

---

## 6. 常见节点类型

### 6.1 图像处理节点

```python
class ImageResize:
    """图像缩放节点"""

    RETURN_TYPES = ("IMAGE",)
    FUNCTION = "resize"
    CATEGORY = "image/transform"

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "image": ("IMAGE", {}),
                "width": ("INT", {"default": 512, "min": 16, "max": 8192, "step": 8}),
                "height": ("INT", {"default": 512, "min": 16, "max": 8192, "step": 8}),
                "keep_ratio": ("BOOLEAN", {"default": False}),
            }
        }

    def resize(self, image, width, height, keep_ratio=False):
        if keep_ratio:
            # 保持宽高比缩放
            orig_w, orig_h = image.shape[2], image.shape[1]
            ratio = min(width / orig_w, height / orig_h)
            new_w = int(orig_w * ratio)
            new_h = int(orig_h * ratio)
            # 对齐到 8 的倍数
            new_w = (new_w // 8) * 8
            new_h = (new_h // 8) * 8
        else:
            new_w, new_h = width, height

        resized = torch.nn.functional.interpolate(
            image.movedim(-1, 1),  # [B, C, H, W]
            size=(new_h, new_w),
            mode="bilinear",
            align_corners=False
        ).movedim(1, -1)  # [B, H, W, C]

        return (resized,)
```

### 6.2 数学运算节点

```python
class MathOperation:
    """数学运算节点"""

    RETURN_TYPES = ("FLOAT",)
    FUNCTION = "calculate"
    CATEGORY = "utils/math"

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "value_a": ("FLOAT", {"default": 0.0}),
                "value_b": ("FLOAT", {"default": 0.0}),
                "operation": (["add", "subtract", "multiply", "divide", "power"], {
                    "default": "add"
                }),
            }
        }

    def calculate(self, value_a, value_b, operation):
        if operation == "add":
            result = value_a + value_b
        elif operation == "subtract":
            result = value_a - value_b
        elif operation == "multiply":
            result = value_a * value_b
        elif operation == "divide":
            result = value_a / value_b if value_b != 0 else 0.0
        elif operation == "power":
            result = value_a ** value_b
        else:
            result = 0.0
        return (result,)
```

### 6.3 文件操作节点

```python
import os
import json
from pathlib import Path

class FileSaver:
    """文件保存节点"""

    RETURN_TYPES = ("STRING",)
    FUNCTION = "save"
    CATEGORY = "utils/file"
    OUTPUT_NODE = True  # 标记为输出节点

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "filename": ("STRING", {"default": "output.txt", "multiline": False}),
                "content": ("STRING", {"default": "", "multiline": True}),
                "directory": ("STRING", {"default": "./output", "multiline": False}),
            }
        }

    def save(self, filename, content, directory):
        # 创建目录
        os.makedirs(directory, exist_ok=True)
        filepath = os.path.join(directory, filename)

        # 写入文件
        with open(filepath, 'w', encoding='utf-8') as f:
            f.write(content)

        return (filepath,)

    @classmethod
    def IS_OUTPUT_NODE(cls):
        return True
```

### 6.4 条件分支节点

```python
class ConditionalRouter:
    """条件路由节点"""

    RETURN_TYPES = ("STRING", "STRING")
    RETURN_NAMES = ("true_output", "false_output")
    FUNCTION = "route"
    CATEGORY = "utils/logic"

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "condition": ("BOOLEAN", {"default": True}),
                "input_a": ("STRING", {"default": "A"}),
                "input_b": ("STRING", {"default": "B"}),
            }
        }

    def route(self, condition, input_a, input_b):
        if condition:
            return (input_a, "")
        else:
            return ("", input_b)
```

### 6.5 批处理节点

```python
class ImageBatchProcessor:
    """图像批处理节点"""

    RETURN_TYPES = ("IMAGE",)
    FUNCTION = "process_batch"
    CATEGORY = "image/batch"
    OUTPUT_IS_LIST = (True,)  # 输出为列表

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "images": ("IMAGE", {}),
                "brightness_adjust": ("FLOAT", {"default": 0.0, "min": -1.0, "max": 1.0, "step": 0.01}),
            }
        }

    def process_batch(self, images, brightness_adjust):
        # 对批量图像进行亮度调整
        adjusted = images + brightness_adjust
        adjusted = torch.clamp(adjusted, 0.0, 1.0)
        return (adjusted,)
```

---

## 7. 高级特性

### 7.1 异步执行

```python
import asyncio

class AsyncNode:
    """异步节点示例"""

    RETURN_TYPES = ("STRING",)
    FUNCTION = "execute"
    CATEGORY = "advanced"

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "text": ("STRING", {"default": "Hello"}),
            }
        }

    async def execute(self, text):
        """异步执行方法"""
        # 模拟异步操作
ls -lh "/home/zhoujun/文档/code/Markdown/AI/comfyui/comfyui创建自定义节点.md" && echo "---" && wc -l "/home/zhoujun/文档/code/Markdown/AI/comfyui/comfyui创建自定义节点.md"
