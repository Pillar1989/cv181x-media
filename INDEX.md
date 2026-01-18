# CV181X-Media Skill 快速导航

> CV181X/CV182X/CV180X 多媒体 API 专家指南 v2.1.0

**最后更新**: 2026-01-18
**适用平台**: Sophgo SG200X 系列 (CV181X/CV182X/CV180X)

---

## 🚀 快速开始

### 新手入门
1. 📖 [SKILL.md](SKILL.md) - 主文档，从这里开始
2. 📋 [README.md](README.md) - 项目概述和版本说明
3. 🔄 [CHANGELOG.md](CHANGELOG.md) - 版本历史和更新记录

### 我想要...

| 需求 | 推荐文档 |
|------|---------|
| **了解整体架构** | [SKILL.md - Module Selection](SKILL.md#1-module-selection-and-architecture-design) |
| **快速配置摄像头** | [VI Module Reference](references/vi.md) - 完整配置流程 |
| **实现视频编码** | [VENC Module Reference](references/venc.md) - H.264/H.265/JPEG |
| **处理视频流** | [VPSS Module Reference](references/vpss.md) - 缩放/旋转/裁剪 |
| **保存/显示视频** | [VO Module Reference](references/vo.md) - LCD/HDMI输出 |
| **调试问题** | [Debug Guide](references/debug.md) - /proc文件系统 |
| **解决错误** | [Troubleshooting](references/troubleshooting.md) - 错误代码和解决方案 |
| **并发场景** | [Concurrent Scenarios](references/concurrent.md) - 多场景设计 |

---

## 📚 模块参考文档

### 视频模块 (Video)

| 模块 | 文档 | 主要功能 |
|------|------|---------|
| **VI** | [vi.md](references/vi.md) | 摄像头输入、ISP管道、DEV/PIPE/CHN架构 |
| **VPSS** | [vpss.md](references/vpss.md) | 视频处理：缩放/旋转/裁剪/格式转换/拼接 |
| **VENC** | [venc.md](references/venc.md) | H.264/H.265/JPEG/MJPEG编码 |
| **VDEC** | [vdec.md](references/vdec.md) | JPEG/MJPEG/H.264解码 |
| **VO** | [vo.md](references/vo.md) | LCD/HDMI显示输出 |

### 音频模块 (Audio)

| 模块 | 文档 | 主要功能 |
|------|------|---------|
| **AI** | [audio.md - Audio Input](references/audio.md#audio-input-ai) | 音频采集、麦克风录音 |
| **AO** | [audio.md - Audio Output](references/audio.md#audio-output-ao) | 音频播放、扬声器输出 |
| **AENC** | [audio.md - Audio Encoder](references/audio.md#audio-encoding-aenc) | 音频编码（PCM/ADPCM/AAC） |
| **ADEC** | [audio.md - Audio Decoder](references/audio.md#audio-decoding-adec) | 音频解码 |
| **VQE** | [audio.md - Voice Quality Enhancement](references/audio.md#voice-quality-enhancement-vqe) | 回声消除、噪声抑制、自动增益 |

### 系统模块 (System)

| 模块 | 文档 | 主要功能 |
|------|------|---------|
| **SYS** | [sys.md](references/sys.md) | 系统控制、模块绑定、内存管理 |
| **VB** | [vb.md](references/vb.md) | 视频缓冲池、Common/Private/User pools |
| **RGN** | [rgn.md](references/rgn.md) | OSD叠加、图形绘制、隐私遮罩 |
| **GDC** | [gdc.md](references/gdc.md) | 几何畸变校正、鱼眼校正、旋转 |

### 工具文档 (Utilities)

| 文档 | 用途 |
|------|------|
| [debug.md](references/debug.md) | /proc文件系统、日志控制、运行时监控 |
| [troubleshooting.md](references/troubleshooting.md) | 错误代码、诊断流程、常见问题 |

---

## 🎯 场景指南

### 完整应用示例

| 场景 | 文档 | 描述 |
|------|------|------|
| 视频监控摄像头 | [scenarios.md - Scenario 1](references/scenarios.md#1-video-surveillance-camera) | VI→VPSS→VENC 完整流程 |
| 智能门铃 | [scenarios.md - Scenario 2](references/scenarios.md#2-smart-doorbell) | 摄像头+显示+编码 |
| 图像处理设备 | [scenarios.md - Scenario 3](references/scenarios.md#3-image-processing-device) | VI→VPSS→自定义处理 |
| 视频会议设备 | [scenarios.md - Scenario 4](references/scenarios.md#4-video-conference-device) | 双向音视频通信 |
| AI视觉相机 | [scenarios.md - Scenario 5](references/scenarios.md#5-ai-powered-security-camera) | TPU推理+结果绘制 |
| 多通道NVR | [scenarios.md - Scenario 6](references/scenarios.md#6-multi-channel-nvr) | 多路视频录制 |

### 并发场景

| 场景 | 文档 | 难度 |
|------|------|------|
| Camera → VPSS (Online) | [concurrent.md - Scenario 1](references/concurrent.md#scenario-1-camera--vpss--savedisplay-online-mode) | ⭐ 基础 |
| File → VENC (Offline) | [concurrent.md - Scenario 2](references/concurrent.md#scenario-2-file--vpss--venc--save-bitstream-offline-mode) | ⭐⭐ 中等 |
| VPSS → TPU Inference | [concurrent.md - Scenario 3](references/concurrent.md#scenario-3-vpss--tpu-inference--draw--save) | ⭐⭐⭐ 高级 |
| 完整并发示例 | [concurrent.md - Scenario 4](references/concurrent.md#scenario-4-complete-concurrent-example) | ⭐⭐⭐⭐ 专家 |

---

## 🔍 故障排除

### 按问题类型查找

#### 初始化问题
- [VB-SYS 初始化顺序](SKILL.md#important-rules) - 必须按顺序初始化
- [VPSS 初始化顺序](references/vpss.md#quick-start) - EnableChn → StartGrp → Bind
- [ERR_VPSS_NOBUF 错误](references/troubleshooting.md#err_vpss_nobuf-0xc006800e) - 缓冲区不足

#### 运行时问题
- [绑定失败](references/troubleshooting.md#binding-issues) - 检查/proc/cvitek/sys
- [帧丢失](references/troubleshooting.md#frame-drops) - 检查VB pool配置
- [性能问题](references/vpss.md#performance-considerations) - VPSS性能优化

#### 内存问题
- [VB Pool 耗尽](references/vb.md#troubleshooting) - 增加buffer数量
- [VENC SendFrame 内存](references/venc.md#sendframe-memory-requirements) - 必须使用VB Pool
- [ION 缓存一致性](SKILL.md#ion-cache-management) - FlushCache/InvalidateCache

### 调试工具

```bash
# 检查模块绑定状态
cat /proc/cvitek/sys | grep -A 10 "BIND RELATION"

# 检查VI状态
cat /proc/cvitek/vi

# 检查VPSS状态
cat /proc/cvitek/vpss

# 检查VB buffer使用
cat /proc/cvitek/vb

# 启用调试日志
echo "VI=7" > /proc/cvitek/log
echo "VPSS=7" > /proc/cvitek/log
```

---

## 🛠️ 自动化工具

### 更新和维护

| 脚本 | 功能 | 使用方法 |
|------|------|---------|
| **update_from_sdk.sh** | 从SDK更新API | `bash scripts/update_from_sdk.sh /path/to/sdk` |
| **learn_from_usage.py** | 学习使用模式 | `python scripts/learn_from_usage.py --feedback-file log.txt` |
| **validate_skill.sh** | 验证skill完整性 | `bash scripts/validate_skill.sh` |

### 快速验证

```bash
# 验证skill完整性
bash scripts/validate_skill.sh

# 预期输出
# Step 1: Checking required files...
#   ✓ SKILL.md
#   ✓ references/vi.md
#   ...
# Status: ✓ PASS (No issues found)
```

---

## 📖 核心概念速查

### Online vs Offline 模式

| 模式 | API | 数据流 | 使用场景 |
|------|-----|--------|---------|
| **Online** | `CVI_SYS_Bind()` | 自动（硬件管理） | 标准视频管道 |
| **Offline** | `GetFrame/SendFrame()` | 手动（CPU参与） | 自定义处理 |

### VPSS 输入源约束

⚠️ **重要**: VPSS Group **不能**动态切换输入源

- **Online Mode**: VI → VPSS (Bind) - 零拷贝
- **Offline Mode**: File/内存 → VPSS (SendFrame) - 手动控制
- **多场景**: 使用**独立的 VPSS Groups**

详见: [VPSS Input Source Constraints](SKILL.md#vpss-input-source-constraints)

### VENC SendFrame 内存要求

⚠️ **关键**: 必须使用 **VB Pool**，不能直接使用 ION 内存

```c
// ✅ 正确
VB_BLK blk = CVI_VB_GetBlock(VB_INVALID_POOLID, size);
frame.u32PoolId = CVI_VB_Handle2PoolId(blk);

// ❌ 错误
CVI_SYS_IonAlloc(&paddr, &vaddr, ...);
frame.u32PoolId = VB_INVALID_POOL_ID;  // 可能失败
```

详见: [VENC SendFrame Memory Requirements](references/venc.md#sendframe-memory-requirements)

### VB Pool 设计

| Pool类型 | 用途 | 配置方式 |
|---------|------|---------|
| **Common Pool** | 共享内存 | `VB_CONFIG_S.astCommPool[]` |
| **Private Pool** | 模块专用 | `CVI_VB_CreatePool()` |
| **EX Mode** | 用户管理 | `VB_POOL_CONFIG_EX_S` |

详见: [VB Module Reference](references/vb.md)

---

## 🔄 版本信息

### 当前版本: v2.1.0 (2026-01-18)

**主要更新**:
- ✅ 源码验证：所有API经过cvi_mpi验证
- ✅ 关键修正：VENC SendFrame内存要求
- ✅ 新增API：SendFrameEx、ION缓存管理、GDC高级功能
- ✅ 并发场景：完整的多场景设计指南（含TPU推理）

**平台支持**:
- CV181X (SG2002) - 全功能支持
- CV182X (SG2002) - 全功能支持
- CV180X (SG2000) - 部分功能限制

### 版本历史

- [v2.1.0](CHANGELOG.md#210---2026-01-18) - 源码验证版本（当前）
- [v2.0.0](CHANGELOG.md#200---2026-01-18) - 完整模块覆盖
- [v1.1.0](CHANGELOG.md#110---2026-01-18) - 故障排除增强
- [v1.0.0](CHANGELOG.md#100---2026-01-17) - 初始版本

---

## 💡 使用技巧

### 1. 按任务快速查找

**任务**: "我想从摄像头采集视频并编码为H.265"

1. 查看 [VI Module](references/vi.md) 配置摄像头
2. 查看 [VPSS Module](references/vpss.md) 处理视频
3. 查看 [VENC Module](references/venc.md) 配置编码
4. 使用 [SYS_Bind](references/sys.md) 连接模块

### 2. 按错误快速查找

**错误**: `ERR_VPSS_NOBUF (0xc006800e)`

1. 查看 [Troubleshooting](references/troubleshooting.md#err_vpss_nobuf-0xc006800e)
2. 检查 [VPSS GetChnFrame](references/vpss.md#offline-mode) 使用
3. 验证 [VB Pool](references/vb.md) 配置

### 3. 学习最佳实践

- 从 [Common Scenarios](references/scenarios.md) 开始
- 参考 [Concurrent Scenarios](references/concurrent.md) 学习高级用法
- 查看 [Debug Guide](references/debug.md) 掌握调试技巧

---

## 📞 获取帮助

### 文档内搜索

使用关键字搜索文档：
```bash
# 在当前目录搜索关键字
grep -r "SendFrame" references/
grep -r "Bind.*VPSS" SKILL.md
```

### 官方资源

- SDK头文件: `/cvi_mpi/include/`
- 示例代码: `/cvi_mpi/sample/`
- 官方文档: SDK PDF手册

### 反馈和贡献

遇到问题或有改进建议？
- 使用 `scripts/learn_from_usage.py` 提供反馈
- 查看 [CHANGELOG.md](CHANGELOG.md) 了解更新历史

---

## 🎓 推荐阅读路径

### 初学者（第一次使用）
1. [README.md](README.md) - 了解项目
2. [SKILL.md - Overview](SKILL.md#overview) - 核心概念
3. [SKILL.md - Quick Start](SKILL.md#quick-start) - 快速入门
4. [scenarios.md - Scenario 1](references/scenarios.md#1-video-surveillance-camera) - 实践示例

### 中级用户（熟悉基础）
1. [concurrent.md - Multi-Scenario](references/concurrent.md) - 并发设计
2. [venc.md - SendFrameEx](references/venc.md#sendframeex-advanced-mode) - 高级编码
3. [vb.md - EX Mode](references/vb.md#vb-pool-ex-mode-usermanaged-blocks) - 内存优化
4. [troubleshooting.md](references/troubleshooting.md) - 问题诊断

### 高级用户（深度优化）
1. [SKILL.md - Performance](SKILL.md#3-performance-optimization) - 性能调优
2. [gdc.md - MESH Management](references/gdc.md#mesh-management-structures) - 高级校正
3. [sys.md - VI/VPSS Working Modes](references/sys.md#vi-vpss-working-modes) - 底层配置
4. [debug.md](references/debug.md) - 深度调试

---

**索引文档版本**: v2.1.0
**最后更新**: 2026-01-18
**维护**: CV181X-Media Skill Team
