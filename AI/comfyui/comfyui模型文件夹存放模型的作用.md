# ComfyUI 模型文件夹完整指南

> 详细说明 ComfyUI 各模型目录的作用、存放模型类型、典型文件及使用建议。
> 
> 最后更新：2026-05-13
---
## 一、快速参考总览

| 目录 | 作用 | 类比 | 是否必须 | 常用度 |
| --- | --- | --- | --- | --- |
| `checkpoints` | 完整 SD 模型（UNet+CLIP+VAE 打包） | 一体机 | 否（Flux 可不用） | ★★★★★ |
| `unet` | 纯扩散模型主体 | 画家 | 是（拆分加载时） | ★★★★★ |
| `diffusion_models` | 新架构扩散模型（Flux/SD3） | 新一代画家 | 是（新架构） | ★★★★★ |
| `clip` | 经典文本编码器（SD1.x/2.x） | 翻译官 | SD 系必须 | ★★★★☆ |
| `text_encoders` | 高级文本编码器（T5/CLIP-L/G） | 高级翻译官 | Flux/SD3 必须 | ★★★★★ |
| `vae` | 图片↔Latent 转换 | 压缩/解压器 | 是 | ★★★★★ |
| `vae_approx` | 快速预览 VAE | 草稿渲染器 | 否 | ★★☆☆☆ |
| `loras` | LoRA 微调模型 | 技能插件 | 否 | ★★★★★ |
| `controlnet` | 条件控制网络 | 导演 | 否 | ★★★★☆ |
| `clip_vision` | 图像编码器 | 看图专家 | 否 | ★★★★☆ |
| `embeddings` | Textual Inversion 向量 | 特殊词典 | 否 | ★★☆☆☆ |
| `hypernetworks` | 超网络（早期微调） | LoRA 前辈 | 否 | ★☆☆☆☆ |
| `style_models` | 风格迁移模型 | 画风参考书 | 否 | ★★☆☆☆ |
| `photomaker` | 人物一致性模型 | 身份保持器 | 否 | ★★☆☆☆ |
| `upscale_models` | 图像超分模型 | 放大镜 | 否 | ★★★★☆ |
| `latent_upscale_models` | Latent 空间超分 | AI 放大镜 | 否 | ★★★☆☆ |
| `detection` | 目标检测模型 | 侦察兵 | 否 | ★★★☆☆ |
| `geometry_estimation` | 深度/法线/姿态估计 | 测绘员 | 否 | ★★★☆☆ |
| `background_removal` | 背景移除/抠图 | 抠图师 | 否 | ★★☆☆☆ |
| `optical_flow` | 光流估计 | 运动分析师 | 否 | ★☆☆☆☆ |
| `frame_interpolation` | 视频补帧 | 补帧师 | 否 | ★☆☆☆☆ |
| `audio_encoders` | 音频编码器 | 听译员 | 特定工作流 | ★☆☆☆☆ |
| `gligen` | 区域控制生成 | 区域导演 | 否 | ★☆☆☆☆ |
| `diffusers` | HF Diffusers 格式模型 | 模型仓库 | 否 | ★★☆☆☆ |
| `configs` | 模型配置文件 | 说明书 | 某些模型需要 | ★★☆☆☆ |
| `model_patches` | 模型补丁 | 热修复包 | 否 | ★☆☆☆☆ |

---
## 二、核心生成模型目录

### 1. `checkpoints` — 一体化检查点

**作用**：存放完整的 Stable Diffusion 检查点，包含 UNet、CLIP 和 VAE 的打包文件。

**典型模型**：
- SD 1.5：`v1-5-pruned.ckpt`、`realisticVisionV51.safetensors`
- SDXL：`sd_xl_base_1.0.safetensors`、`juggernautXL_v9Rundiffusion.safetensors`
- SD 3：`sd3_medium.safetensors`

**加载方式**：`CheckpointLoader` 节点直接加载。

**注意**：Flux 架构通常不使用此目录，而是拆分为 `unet` + `text_encoders` + `vae`。

---

### 2. `unet` — 纯扩散模型

**作用**：存放纯 UNet 扩散模型，不含文本编码器和 VAE。

**典型模型**：
- SDXL UNet：`sdxl_unet.safetensors`
- Flux UNet：`flux1-dev.safetensors`、`flux1-schnell.safetensors`

**加载方式**：通过 `UNETLoader` 加载，配合独立的 text_encoders 和 vae。

---

### 3. `diffusion_models` — 新架构扩散模型

**作用**：存放采用 Transformer/DiT 等新架构的扩散模型。

**典型模型**：
- Flux：`flux1-dev-fp8.safetensors`、`flux1-dev-bf16.safetensors`
- SD3：`sd3.5_large.safetensors`、`sd3.5_large_turbo.safetensors`

**加载方式**：`UNETLoader` + `DualCLIPLoader` 或专用节点。

---

### 4. `clip` — 经典文本编码器

**作用**：存放 SD 1.x/2.x 使用的 CLIP 文本编码器。

**典型模型**：
- `clip_vit_large_14.safetensors`
- `openclip_model.safetensors`

**注意**：SDXL 和 Flux 使用 `text_encoders` 目录下的新格式。

---

### 5. `text_encoders` — 高级文本编码器

**作用**：存放新一代文本编码器，支持多模态和更大上下文。

**典型模型**：
- CLIP-L：`clip_l.safetensors`（SDXL/Flux 需要）
- CLIP-G：`clip_g.safetensors`（SDXL/Flux 需要）
- T5-XXL：`umt5_xxl_fp8_e4m3fn.safetensors`（SD3/Flux 需要）

**加载组合**：
- Flux：CLIP-L + CLIP-G
- SD3：CLIP-L + CLIP-G + T5-XXL
- SDXL：CLIP-L + CLIP-G

---

### 6. `vae` — 变分自编码器

**作用**：负责像素空间与 Latent 空间之间的编码/解码。

**典型模型**：
- SD 1.5：`vae-ft-mse-840000-ema.safetensors`
- SDXL：`sdxl_vae.safetensors`
- Flux：`ae.safetensors`

**注意**：若 checkpoint 已内置 VAE，可跳过独立加载。

---

### 7. `loras` — LoRA 微调模型

**作用**：轻量级微调模型，调整风格、角色或细节。

**典型模型**：
- 风格 LoRA：`anime_style_lora.safetensors`
- 角色 LoRA：`character_lora.safetensors`
- 细节增强：`detail_tuner.safetensors`

**加载方式**：`LoraLoader` 或 `LoraLoaderModelOnly`，可叠加多个。

**注意**：不同架构的 LoRA 不通用（SD1.5 LoRA 不能用于 SDXL）。

---

### 8. `controlnet` — 条件控制网络

**作用**：提供结构、姿态、边缘等条件控制。

**典型模型**：
- Canny：`control_v11p_sd15_canny.safetensors`
- Depth：`control_v11f1p_sd15_depth.safetensors`
- OpenPose：`control_v11p_sd15_openpose.safetensors`
- SDXL：`control-lora-canny-rank128.safetensors`
- Flux：`flux-controlnet-v0.safetensors`

**加载方式**：`ControlNetLoader` + `ControlNetApply` + 预处理节点。

---

### 9. `clip_vision` — 视觉编码器

**作用**：图像理解，用于 IP-Adapter、图像提示等。

**典型模型**：
- `clip_vision_vit_h.safetensors`
- `sigclip_vision_patch14_384.safetensors`
- `clip_vision_g.safetensors`

**使用场景**：IP-Adapter、Image Prompt、ReVision。

---

## 三、辅助/功能型目录

### `embeddings` — Textual Inversion 嵌入
小型嵌入模型（几 MB），添加自定义概念或修复问题。
- 典型：`bad-hands-5.pt`、`easy-negative-hand.pt`

### `hypernetworks` — 超网络
早期微调技术，已被 LoRA 取代，保留兼容性。

### `style_models` — 风格模型
StyleGAN 转换或特定风格迁移模型。

### `photomaker` — 人物一致性
保持多张图中人物身份一致性。

### `upscale_models` — 图像超分
- `RealESRGAN_x4plus.pth`
- `Swin2SR_x4.safetensors`
- `4x-UltraSharp.pth`

### `latent_upscale_models` — Latent 超分
Latent 空间直接放大，如 `4x-ninetailed.pth`。

### `detection` — 目标检测
- 人脸检测、手部检测、人体关键点

### `geometry_estimation` — 几何估计
- 深度图、法线图、姿态估计模型

### `background_removal` — 背景移除
- 抠图模型，如 `RMBG-1.4`

### `optical_flow` — 光流估计
- 视频帧间运动分析

### `frame_interpolation` — 视频补帧
- RIFE、AMT 等补帧模型

### `audio_encoders` — 音频编码器
- 音频到图像的跨模态工作流

### `gligen` — 区域控制
- 控制生成内容在图像中的具体位置

### `diffusers` — Diffusers 格式
- HuggingFace Diffusers 格式的完整模型目录

### `configs` — 配置文件
- `.yaml` 配置文件，某些模型需要

### `model_patches` — 模型补丁
- 模型的热修复或增强补丁

---

## 四、扩展插件相关目录

### `ipadapter` — IP-Adapter 模型
- `ip-adapter-plus_sd15.safetensors`
- `ip-adapter-plus-face_sd15.safetensors`
- `ip-adapter-plus-face_sdxl_vit-h.safetensors`

### `instantid` — InstantID 模型
- `instantid-ip-adapter.safetensors`

### `sams` — Segment Anything 模型
- `sam_vit_h_4b8939.pth`
- `sam_vit_l_0b3195.pth`

### `blip` — BLIP 图像描述模型
- `model_base_caption.pt`
- `model_large_caption.pt`

### `facial_recognition` — 人脸识别
- `antelopev2` 目录下的模型文件

### `mmdets` — MMDetection 模型
- 目标检测配置文件和权重

### `annotators` — 标注器模型
- ControlNet 预处理所需的标注模型

### `srgp` — SRGP 模型
- 特定工作流使用的模型

---

## 五、目录结构建议

ComfyUI/models/  
├── checkpoints/ # 完整检查点（SD1.5/SDXL/SD3）  
├── unet/ # 纯 UNet 模型  
├── diffusion_models/ # 新架构扩散模型（Flux/SD3）  
├── text_encoders/ # 文本编码器（CLIP-L/G, T5）  
├── clip/ # 经典 CLIP（SD1.x/2.x）  
├── vae/ # VAE 模型  
├── loras/ # LoRA 微调  
├── controlnet/ # ControlNet 模型  
├── clip_vision/ # 视觉编码器  
├── upscale_models/ # 图像超分  
├── embeddings/ # 嵌入向量  
├── ipadapter/ # IP-Adapter  
├── sams/ # SAM 模型  
├── detection/ # 检测模型  
├── geometry_estimation/ # 几何估计  
└── configs/ # 配置文件


---

## 六、架构加载对照表

| 架构 | checkpoints | unet/diffusion_models | text_encoders | vae |
| --- | --- | --- | --- | --- |
| SD 1.5 | ✅ 直接加载 | ❌ | ❌（内置） | ⚠️ 可选独立 |
| SD 2.x | ✅ 直接加载 | ❌ | ❌（内置） | ⚠️ 可选独立 |
| SDXL | ✅ 直接加载 | ❌ | ❌（内置） | ⚠️ 可选独立 |
| SD3 | ❌ | ✅ diffusion_models | ✅ CLIP-L+G+T5 | ✅ ae |
| Flux | ❌ | ✅ diffusion_models | ✅ CLIP-L+G | ✅ ae |

---

## 七、最佳实践

### 1. 模型命名规范
{模型名}_{版本}_{架构}_{权重格式}.{扩展名}  
示例：flux1-dev-fp8.safetensors

### 2. 格式优先级
1. `.safetensors` — 推荐，安全无代码执行风险
2. `.ckpt` — 老格式，SD 1.5 常见
3. `.pt` / `.pth` — PyTorch 原生格式

### 3. 显存管理
- T5-XXL 约需 10GB+ 显存
- SDXL 约需 8-10GB
- Flux dev 约需 12-16GB
- 使用 fp8 量化可显著降低显存占用

### 4. 注意事项
- **架构隔离**：不同架构的模型/LoRA 不通用
- **依赖检查**：新模型可能需要特定自定义节点
- **版本兼容**：ComfyUI 更新后旧节点可能失效
- **备份重要**：定期备份自定义工作流和模型列表

---

*文档维护：洛微雅 (Roviya)*
