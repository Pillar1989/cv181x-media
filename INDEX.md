# CV181X-Media Skill Quick Navigation

> Expert Guide for CV181X/CV182X/CV180X Multimedia API v2.1.0

**Last Updated**: 2026-01-18
**Supported Platforms**: Sophgo SG200X Series (CV181X/CV182X/CV180X)

---

## 🚀 Quick Start

### For Beginners
1. 📖 [SKILL.md](SKILL.md) - Main documentation, start here
2. 📋 [README.md](README.md) - Project overview and version information
3. 🔄 [CHANGELOG.md](CHANGELOG.md) - Version history and update records

### I Want To...

| Requirement | Recommended Documentation |
|------|---------|
| **Understand overall architecture** | [SKILL.md - Module Selection](SKILL.md#1-module-selection-and-architecture-design) |
| **Quickly configure camera** | [VI Module Reference](references/vi.md) - Complete configuration workflow |
| **Implement video encoding** | [VENC Module Reference](references/venc.md) - H.264/H.265/JPEG |
| **Process video streams** | [VPSS Module Reference](references/vpss.md) - Scaling/Rotation/Cropping |
| **Save/Display video** | [VO Module Reference](references/vo.md) - LCD/HDMI output |
| **Debug issues** | [Debug Guide](references/debug.md) - /proc filesystem |
| **Resolve errors** | [Troubleshooting](references/troubleshooting.md) - Error codes and solutions |
| **Concurrent scenarios** | [Concurrent Scenarios](references/concurrent.md) - Multi-scenario design |

---

## 📚 Module Reference Documentation

### Video Modules

| Module | Documentation | Main Features |
|------|------|---------|
| **VI** | [vi.md](references/vi.md) | Camera input, ISP pipeline, DEV/PIPE/CHN architecture |
| **VPSS** | [vpss.md](references/vpss.md) | Video processing: scaling/rotation/cropping/format conversion/stitching |
| **VENC** | [venc.md](references/venc.md) | H.264/H.265/JPEG/MJPEG encoding |
| **VDEC** | [vdec.md](references/vdec.md) | JPEG/MJPEG/H.264 decoding |
| **VO** | [vo.md](references/vo.md) | LCD/HDMI display output |

### Audio Modules

| Module | Documentation | Main Features |
|------|------|---------|
| **AI** | [audio.md - Audio Input](references/audio.md#audio-input-ai) | Audio capture, microphone recording |
| **AO** | [audio.md - Audio Output](references/audio.md#audio-output-ao) | Audio playback, speaker output |
| **AENC** | [audio.md - Audio Encoder](references/audio.md#audio-encoding-aenc) | Audio encoding (PCM/ADPCM/AAC) |
| **ADEC** | [audio.md - Audio Decoder](references/audio.md#audio-decoding-adec) | Audio decoding |
| **VQE** | [audio.md - Voice Quality Enhancement](references/audio.md#voice-quality-enhancement-vqe) | Echo cancellation, noise suppression, auto gain |

### System Modules

| Module | Documentation | Main Features |
|------|------|---------|
| **SYS** | [sys.md](references/sys.md) | System control, module binding, memory management |
| **VB** | [vb.md](references/vb.md) | Video buffer pool, Common/Private/User pools |
| **RGN** | [rgn.md](references/rgn.md) | OSD overlay, graphics drawing, privacy masking |
| **GDC** | [gdc.md](references/gdc.md) | Geometric distortion correction, fisheye correction, rotation |

### Utility Documentation

| Documentation | Purpose |
|------|------|
| [debug.md](references/debug.md) | /proc filesystem, log control, runtime monitoring |
| [troubleshooting.md](references/troubleshooting.md) | Error codes, diagnostic procedures, common issues |

---

## 🎯 Scenario Guides

### Complete Application Examples

| Scenario | Documentation | Description |
|------|------|------|
| Video Surveillance Camera | [scenarios.md - Scenario 1](references/scenarios.md#1-video-surveillance-camera) | VI→VPSS→VENC complete workflow |
| Smart Doorbell | [scenarios.md - Scenario 2](references/scenarios.md#2-smart-doorbell) | Camera + display + encoding |
| Image Processing Device | [scenarios.md - Scenario 3](references/scenarios.md#3-image-processing-device) | VI→VPSS→custom processing |
| Video Conference Device | [scenarios.md - Scenario 4](references/scenarios.md#4-video-conference-device) | Bidirectional audio/video communication |
| AI Vision Camera | [scenarios.md - Scenario 5](references/scenarios.md#5-ai-powered-security-camera) | TPU inference + result drawing |
| Multi-channel NVR | [scenarios.md - Scenario 6](references/scenarios.md#6-multi-channel-nvr) | Multi-channel video recording |

### Concurrent Scenarios

| Scenario | Documentation | Difficulty |
|------|------|------|
| Camera → VPSS (Online) | [concurrent.md - Scenario 1](references/concurrent.md#scenario-1-camera--vpss--savedisplay-online-mode) | ⭐ Basic |
| File → VENC (Offline) | [concurrent.md - Scenario 2](references/concurrent.md#scenario-2-file--vpss--venc--save-bitstream-offline-mode) | ⭐⭐ Intermediate |
| VPSS → TPU Inference | [concurrent.md - Scenario 3](references/concurrent.md#scenario-3-vpss--tpu-inference--draw--save) | ⭐⭐⭐ Advanced |
| Complete Concurrent Example | [concurrent.md - Scenario 4](references/concurrent.md#scenario-4-complete-concurrent-example) | ⭐⭐⭐⭐ Expert |

---

## 🔍 Troubleshooting

### Find by Issue Type

#### Initialization Issues
- [VB-SYS Initialization Order](SKILL.md#important-rules) - Must initialize in sequence
- [VPSS Initialization Order](references/vpss.md#quick-start) - EnableChn → StartGrp → Bind
- [ERR_VPSS_NOBUF Error](references/troubleshooting.md#err_vpss_nobuf-0xc006800e) - Insufficient buffers

#### Runtime Issues
- [Binding Failure](references/troubleshooting.md#binding-issues) - Check /proc/cvitek/sys
- [Frame Drops](references/troubleshooting.md#frame-drops) - Check VB pool configuration
- [Performance Issues](references/vpss.md#performance-considerations) - VPSS performance optimization

#### Memory Issues
- [VB Pool Exhaustion](references/vb.md#troubleshooting) - Increase buffer count
- [VENC SendFrame Memory](references/venc.md#sendframe-memory-requirements) - Must use VB Pool
- [ION Cache Coherence](SKILL.md#ion-cache-management) - FlushCache/InvalidateCache

### Debugging Tools

```bash
# Check module binding status
cat /proc/cvitek/sys | grep -A 10 "BIND RELATION"

# Check VI status
cat /proc/cvitek/vi

# Check VPSS status
cat /proc/cvitek/vpss

# Check VB buffer usage
cat /proc/cvitek/vb

# Enable debug logs
echo "VI=7" > /proc/cvitek/log
echo "VPSS=7" > /proc/cvitek/log
```

---

## 🛠️ Automation Tools

### Update and Maintenance

| Script | Function | Usage |
|------|------|---------|
| **update_from_sdk.sh** | Update APIs from SDK | `bash scripts/update_from_sdk.sh /path/to/sdk` |
| **learn_from_usage.py** | Learn usage patterns | `python scripts/learn_from_usage.py --feedback-file log.txt` |
| **validate_skill.sh** | Validate skill integrity | `bash scripts/validate_skill.sh` |

### Quick Validation

```bash
# Validate skill integrity
bash scripts/validate_skill.sh

# Expected output
# Step 1: Checking required files...
#   ✓ SKILL.md
#   ✓ references/vi.md
#   ...
# Status: ✓ PASS (No issues found)
```

---

## 📖 Core Concepts Quick Reference

### Online vs Offline Mode

| Mode | API | Data Flow | Use Cases |
|------|-----|--------|---------|
| **Online** | `CVI_SYS_Bind()` | Automatic (hardware-managed) | Standard video pipeline |
| **Offline** | `GetFrame/SendFrame()` | Manual (CPU-involved) | Custom processing |

### VPSS Input Source Constraints

⚠️ **Important**: VPSS Group **cannot** dynamically switch input sources

- **Online Mode**: VI → VPSS (Bind) - Zero-copy
- **Offline Mode**: File/Memory → VPSS (SendFrame) - Manual control
- **Multi-scenario**: Use **separate VPSS Groups**

See: [VPSS Input Source Constraints](SKILL.md#vpss-input-source-constraints)

### VENC SendFrame Memory Requirements

⚠️ **Critical**: Must use **VB Pool**, cannot use ION memory directly

```c
// ✅ Correct
VB_BLK blk = CVI_VB_GetBlock(VB_INVALID_POOLID, size);
frame.u32PoolId = CVI_VB_Handle2PoolId(blk);

// ❌ Incorrect
CVI_SYS_IonAlloc(&paddr, &vaddr, ...);
frame.u32PoolId = VB_INVALID_POOL_ID;  // May fail
```

See: [VENC SendFrame Memory Requirements](references/venc.md#sendframe-memory-requirements)

### VB Pool Design

| Pool Type | Purpose | Configuration |
|---------|------|---------|
| **Common Pool** | Shared memory | `VB_CONFIG_S.astCommPool[]` |
| **Private Pool** | Module-specific | `CVI_VB_CreatePool()` |
| **EX Mode** | User-managed | `VB_POOL_CONFIG_EX_S` |

See: [VB Module Reference](references/vb.md)

---

## 🔄 Version Information

### Current Version: v2.1.0 (2026-01-18)

**Major Updates**:
- ✅ Source verification: All APIs verified against cvi_mpi
- ✅ Critical fix: VENC SendFrame memory requirements
- ✅ New APIs: SendFrameEx, ION cache management, GDC advanced features
- ✅ Concurrent scenarios: Complete multi-scenario design guide (including TPU inference)

**Platform Support**:
- CV181X (SG2002) - Full feature support
- CV182X (SG2002) - Full feature support
- CV180X (SG2000) - Partial feature limitations

### Version History

- [v2.1.0](CHANGELOG.md#210---2026-01-18) - Source verification version (current)
- [v2.0.0](CHANGELOG.md#200---2026-01-18) - Complete module coverage
- [v1.1.0](CHANGELOG.md#110---2026-01-18) - Enhanced troubleshooting
- [v1.0.0](CHANGELOG.md#100---2026-01-17) - Initial version

---

## 💡 Usage Tips

### 1. Quick Find by Task

**Task**: "I want to capture video from camera and encode as H.265"

1. Check [VI Module](references/vi.md) to configure camera
2. Check [VPSS Module](references/vpss.md) to process video
3. Check [VENC Module](references/venc.md) to configure encoding
4. Use [SYS_Bind](references/sys.md) to connect modules

### 2. Quick Find by Error

**Error**: `ERR_VPSS_NOBUF (0xc006800e)`

1. Check [Troubleshooting](references/troubleshooting.md#err_vpss_nobuf-0xc006800e)
2. Check [VPSS GetChnFrame](references/vpss.md#offline-mode) usage
3. Verify [VB Pool](references/vb.md) configuration

### 3. Learn Best Practices

- Start with [Common Scenarios](references/scenarios.md)
- Reference [Concurrent Scenarios](references/concurrent.md) for advanced usage
- Check [Debug Guide](references/debug.md) for debugging techniques

---

## 📞 Get Help

### Search Within Documentation

Use keywords to search documents:
```bash
# Search keywords in current directory
grep -r "SendFrame" references/
grep -r "Bind.*VPSS" SKILL.md
```

### Official Resources

- SDK headers: `/cvi_mpi/include/`
- Sample code: `/cvi_mpi/sample/`
- Official documentation: SDK PDF manual

### Feedback and Contribution

Encountered issues or have improvement suggestions?
- Use `scripts/learn_from_usage.py` to provide feedback
- Check [CHANGELOG.md](CHANGELOG.md) for update history

---

## 🎓 Recommended Reading Paths

### Beginners (First Time Users)
1. [README.md](README.md) - Understand the project
2. [SKILL.md - Overview](SKILL.md#overview) - Core concepts
3. [SKILL.md - Quick Start](SKILL.md#quick-start) - Quick start guide
4. [scenarios.md - Scenario 1](references/scenarios.md#1-video-surveillance-camera) - Practical example

### Intermediate Users (Familiar with Basics)
1. [concurrent.md - Multi-Scenario](references/concurrent.md) - Concurrent design
2. [venc.md - SendFrameEx](references/venc.md#sendframeex-advanced-mode) - Advanced encoding
3. [vb.md - EX Mode](references/vb.md#vb-pool-ex-mode-usermanaged-blocks) - Memory optimization
4. [troubleshooting.md](references/troubleshooting.md) - Problem diagnosis

### Advanced Users (Deep Optimization)
1. [SKILL.md - Performance](SKILL.md#3-performance-optimization) - Performance tuning
2. [gdc.md - MESH Management](references/gdc.md#mesh-management-structures) - Advanced correction
3. [sys.md - VI/VPSS Working Modes](references/sys.md#vi-vpss-working-modes) - Low-level configuration
4. [debug.md](references/debug.md) - Deep debugging

---

**Index Document Version**: v2.1.0
**Last Updated**: 2026-01-18
**Maintained by**: CV181X-Media Skill Team
