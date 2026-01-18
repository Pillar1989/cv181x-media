---
name: cv181x-media
description: "Expert guidance for CV181X/CV182X/CV180X multimedia API development on Sophgo platforms (SG200X series). Provides comprehensive knowledge of VI (Video Input), VPSS (Video Processing), VENC (Video Encoding), VDEC (Video Decoding), VO (Video Output), Audio (AI/AO/AENC/ADEC/VQE), SYS (System Control), VB (Video Buffer Pool), REGION (Regional Management/OSD), and GDC (Geometric Distortion Correction Subsystem) modules. Use this skill when working with: (1) Video capture from camera sensors via MIPI/LVDS/HISPI/SLVS/BT.1120/BT.656/BT.601, (2) Video encoding (H.264/H.265/JPEG/MJPEG) with ROI, GOP, frame skipping, (3) Video decoding (JPEG/MJPEG/H.264), (4) Video processing (scaling, rotation, cropping, format conversion, stitching), (5) Video display output (CV181X only), (6) On-screen display (OSD) and graphics overlay, (7) Audio capture, playback, encoding, decoding, voice enhancement, (8) Fisheye correction and lens distortion correction, (9) Module binding and system integration, (10) Video buffer memory management, (11) System monitoring (temperature, thermal callbacks), (12) Deep learning pre-processing and TPU integration, (13) Dual-OS communication, (14) Debugging multimedia applications, (15) Building multimedia applications (surveillance cameras, video conferencing, AI vision systems) on CV181X/CV182X/CV180X platforms."
---

# CV181X/CV182X/CV180X Multimedia API Expert

## Overview

This skill provides expert guidance for developing multimedia applications on Sophgo CV181X/CV182X/CV180X platforms (SG200X series) using the CVI MPI (Media Processing Interface) API. It covers video capture, processing, encoding, decoding, output, audio processing, and system integration for embedded multimedia applications.

## Core Capabilities

### 1. Module Selection and Architecture Design

When designing multimedia applications, choose the right combination of modules:

**Video Modules**:
- **VI (Video Input)**: Camera sensor input, ISP pipeline (DEV/ISP_FE/ISP_BE/CHN)
- **VPSS (Video Processing)**: Scaling, cropping, rotation, format conversion, stitching
- **VENC (Video Encoding)**: H.264/H.265/JPEG/MJPEG encoding
- **VDEC (Video Decoding)**: JPEG/MJPEG/H.264 decoding (CV181X only)
- **VO (Video Output)**: LCD/HDMI display (CV181X only, CV180X not supported)

**Audio Modules**:
- **AI (Audio Input)**: Audio capture from microphone
- **AO (Audio Output)**: Audio playback to speaker
- **AENC (Audio Encoding)**: Audio encoding (PCM/ADPCM/AAC)
- **ADEC (Audio Decoding)**: Audio decoding
- **VQE (Voice Quality Enhancement)**: AEC, ANR, AGC for speech quality

**System Modules**:
- **SYS (System Control)**: Module binding, memory management, temperature monitoring
- **VB (Video Buffer Pool)**: Unified video memory management
- **REGION (Regional Management)**: OSD, graphics overlay, privacy masking (RGN is abbreviation)
- **GDC (Geometric Distortion Correction Subsystem)**: Lens correction, fisheye dewarp, rotation

**Decision Framework**:

1. **What is the data source?**
   - Camera sensor → Use VI
   - Network stream → Use VDEC (decoder)
   - File/Memory → Use manual frame operations

2. **What processing is needed?**
   - Resolution change → Use VPSS
   - Rotation/Crop → Use VPSS or VI (VI has limited support)
   - Format conversion → Use VPSS
   - None → Skip VPSS, bind VI directly to VENC/VO

3. **What is the output?**
   - Network/File → Use VENC
   - Display → Use VO
   - Custom processing → Use GetFrame/ReleaseFrame

4. **How should modules connect?**
   - Real-time, low latency → Use online mode (SYS_Bind)
   - Custom processing → Use offline mode (GetFrame/SendFrame)

**Common Pipelines**:
```
Video Surveillance:  VI → VPSS → VENC → Network
Video Doorbell:      VI → VPSS → VO (display) + VENC (record)
AI Vision:           VI → VPSS → User (TPU) → VPSS → VENC/VO
Video Conference:    VI → VPSS → VENC (send) + VDEC → VPSS → VO (receive)
```

For detailed scenarios, see [references/scenarios.md](references/scenarios.md).

### 2. Module Configuration Workflow

**Standard Initialization Pattern** (applies to all modules):

1. **Initialize system**: `CVI_SYS_Init()`
2. **Configure module**: Set attributes (resolution, format, etc.)
3. **Enable/Start module**: Enable device/channel, start processing
4. **Bind modules**: Connect modules for automatic data flow (optional)
5. **Processing**: Automatic (online) or manual (offline)
6. **Cleanup**: Unbind → Disable/Stop → Destroy

**Online Mode (Recommended)**:
```
CVI_SYS_Init()
→ Configure all modules (VI, VPSS, VENC)
→ Start all modules
→ Bind modules: CVI_SYS_Bind(src, dest)
→ Automatic data flow (no manual frame handling)
→ Only retrieve encoded bitstream from VENC
```

**Offline Mode (Advanced)**:
```
CVI_SYS_Init()
→ Configure modules
→ Start modules
→ Loop:
  - GetFrame from source
  - Process frame (custom algorithm)
  - SendFrame to destination
  - ReleaseFrame
```

### 3. Video Input (VI) Operations

**When to consult**: Camera sensor integration, video capture, ISP configuration

**Key Operations**:
- Configure sensor interface (MIPI, DVP)
- Setup ISP pipeline (pipe configuration)
- Enable output channels (multiple resolutions)
- Crop, rotate, flip operations
- Bind to VPSS/VENC/VO

**Quick Start**:
```
1. CVI_VI_SetDevAttr() - Configure sensor interface
2. CVI_VI_EnableDev() - Enable device
3. CVI_VI_CreatePipe() - Create ISP pipe
4. CVI_VI_SetPipeAttr() - Configure pipe (resolution, format)
5. CVI_VI_StartPipe() - Start processing
6. CVI_VI_SetChnAttr() - Configure channel (output resolution)
7. CVI_VI_EnableChn() - Enable channel
8. CVI_SYS_Bind() - Bind to next module
```

**Reference**: See [references/vi.md](references/vi.md) for complete API list and workflows.

### 4. Video Processing (VPSS) Operations

**When to consult**: Scaling, cropping, rotation, format conversion, image enhancement

**Key Operations**:
- Multi-resolution output (1 input → up to 4 outputs)
- Scaling (up to 16x upscale, 1/32 downscale)
- Rotation (0/90/180/270)
- Cropping (group-level and channel-level)
- Format conversion (YUV420/YUV422/RGB)
- Image enhancement (brightness, contrast, saturation, hue)

**Quick Start** (order is CRITICAL - from official SDK sample):
```
1. CVI_VPSS_CreateGrp() - Create VPSS group
2. CVI_VPSS_ResetGrp() - Reset group (REQUIRED!)
3. CVI_VPSS_SetChnAttr() - Set output size and format (per channel)
4. CVI_VPSS_EnableChn() - Enable channels FIRST
5. CVI_VPSS_StartGrp() - Start group AFTER enable
6. CVI_SYS_Bind() - Bind LAST (after both VI and VPSS are started)
```

**CRITICAL**: The order EnableChn → StartGrp → Bind is mandatory. Binding before StartGrp will silently fail.

**Multi-Resolution Example**:
```c
// Create group for 1080p input
CVI_VPSS_CreateGrp(VpssGrp, &grpAttr);  // Input: 1920x1080

// Setup 3 output channels
SetChnAttr(VpssGrp, 0, 1920, 1080);    // Chn0: 1080p (main stream)
SetChnAttr(VpssGrp, 1, 1280, 720);     // Chn1: 720p (sub stream)
SetChnAttr(VpssGrp, 2, 640, 360);      // Chn2: 360p (mobile stream)
```

**Reference**: See [references/vpss.md](references/vpss.md) for complete API list and performance considerations.

### 5. Video Encoding (VENC) Operations

**When to consult**: H.264/H.265/JPEG/MJPEG encoding, bitstream generation, rate control

**Key Operations**:
- Create encoding channel (H.264/H.265/JPEG/MJPEG)
- Configure rate control (CBR, VBR, FIXQP)
- Set GOP structure
- Retrieve encoded bitstream
- ROI encoding (better quality for specific regions)
- Force IDR frames

**Quick Start**:
```
1. CVI_VENC_CreateChn() - Create channel with codec type
2. CVI_VENC_SetRcParam() - Set bitrate and rate control mode
3. CVI_VENC_StartRecvFrame() - Start accepting frames
4. CVI_SYS_Bind() - Bind from VI/VPSS
5. Loop:
   - CVI_VENC_GetStream() - Get encoded bitstream
   - Process/Save bitstream
   - CVI_VENC_ReleaseStream() - Release buffer
```

**Rate Control Modes**:
- **CBR** (Constant Bitrate): Best for streaming (stable bandwidth)
- **VBR** (Variable Bitrate): Best for storage (better quality)
- **FIXQP** (Fixed QP): Best for quality testing

**Reference**: See [references/venc.md](references/venc.md) for codec configuration and bitrate guidelines.

### 6. Video Output (VO) Operations

**When to consult**: LCD/HDMI display, video overlay, screen output

**Key Operations**:
- Configure display interface (MIPI DSI, I8080, RGB)
- Setup video layer and channels
- Multi-window display
- Rotation and mirroring
- Image enhancement (brightness, contrast, gamma)

**Quick Start**:
```
1. CVI_VO_SetPubAttr() - Set interface type and resolution
2. CVI_VO_Enable() - Enable device
3. CVI_VO_SetVideoLayerAttr() - Configure layer
4. CVI_VO_EnableVideoLayer() - Enable layer
5. CVI_VO_SetChnAttr() - Configure channel (window position/size)
6. CVI_VO_EnableChn() - Enable channel
7. CVI_SYS_Bind() - Bind from VI/VPSS
```

**Reference**: See [references/vo.md](references/vo.md) for display configuration and multi-channel setup.

### 7. System Integration (SYS) Operations

**When to consult**: Module binding, memory management, system configuration

**Key Operations**:
- Initialize/cleanup media system
- Bind modules for zero-copy data flow
- Allocate ION memory for custom buffers
- Configure VI/VPSS working modes (online/offline)
- DMA memory copy

**Module Binding Pattern**:
```c
// Define source (VI channel)
MMF_CHN_S stSrcChn = {.enModId = CVI_ID_VI, .s32DevId = ViPipe, .s32ChnId = ViChn};

// Define destination (VPSS group)
// IMPORTANT: For VPSS as destination, s32ChnId MUST be 0 (per official documentation)
MMF_CHN_S stDestChn = {.enModId = CVI_ID_VPSS, .s32DevId = VpssGrp, .s32ChnId = 0};

// Bind modules (MUST be done AFTER modules are configured and started)
// CRITICAL: Both VI (EnableChn) and VPSS (StartGrp) must be complete before binding
CVI_SYS_Bind(&stSrcChn, &stDestChn);

// Data flows automatically from VI to VPSS
// Verify binding: cat /proc/cvitek/sys | grep -A 10 "BIND RELATION"
// ...

// Cleanup: Unbind BEFORE stopping modules
CVI_SYS_UnBind(&stSrcChn, &stDestChn);
```

**System Binding Mechanism**:

The SDK provides system binding interfaces to establish relationships between data sources and receivers:

- **After binding**: Data from source automatically flows to destination (zero-copy)
- **One-to-many**: A single source can bind to multiple destinations
- **Automatic return**: If source is unbound, data automatically returns to VB pool
- **Hardware-managed**: Zero-copy data transfer managed by hardware

**Complete Binding Table**:

| Data Source      | Valid Data Receivers                            |
|------------------|-------------------------------------------------|
| **VI**           | VPSS, VENC, VO                                  |
| **VDEC**         | VPSS → (VENC, VO)                               |
| **Audio Input**  | AENC                                            |
| **Audio Output** | ADEC → Audio Output                             |

**Key Binding Patterns**:
```
VI              →  VPSS
                    →  VENC
                    →  VO

VDEC            →  VPSS
                    →  VENC
                    →  VO

Audio Input     →  AENC

Audio Output    →  ADEC
                    →  Audio Output
```

**One-to-Many Binding Example**:
```c
// One VI source feeding multiple outputs
MMF_CHN_S vi_chn = {CVI_ID_VI, 0, 0};
MMF_CHN_S vpss_chn = {CVI_ID_VPSS, 0, 0};
MMF_CHN_S venc_chn = {CVI_ID_VENC, 0, 0};
MMF_CHN_S vo_chn = {CVI_ID_VO, 0, 0};

// Bind VI to both VPSS and VENC
CVI_SYS_Bind(&vi_chn, &vpss_chn);   // VI → VPSS → VO (display)
CVI_SYS_Bind(&vi_chn, &venc_chn);   // VI → VENC (record)
```

**Binding Rules (from official documentation)**:
- When VPSS is the **receiver**, set `s32ChnId = 0` (group receives, not channel)
- When VPSS is the **sender**, set `s32ChnId = VpssChn` (output channel)
- Binding must occur AFTER both source and destination modules are started
- An empty binding table in `/proc/cvitek/sys` indicates binding failed

**Important Rules**:
- Always call `CVI_SYS_Init()` first
- Bind modules AFTER they are configured and started
- Unbind BEFORE stopping/destroying modules
- Always call `CVI_SYS_Exit()` when shutting down

**Reference**: See [references/sys.md](references/sys.md) for binding patterns and memory management.

### 8. Video Buffer Pool (VB) Operations

**When to consult**: Memory allocation, buffer pool configuration, VB pool exhaustion

**Key Operations**:
- Configure common buffer pools (shared by all modules)
- Create private pools for specific modules
- Monitor buffer usage and tune pool sizes
- Troubleshoot "Out of buffers" errors

**Quick Start**:
```
1. CVI_VB_SetConfig() - Configure pools (BEFORE CVI_VB_Init)
2. CVI_VB_Init() - Initialize and allocate pools
3. CVI_SYS_Init() - Initialize system (uses VB pools)
4. Modules automatically use VB pools
5. Monitor: cat /proc/cvitek/vb
```

**Buffer Size Calculation**:
- YUV420: Width × Height × 3 / 2
- YUV422: Width × Height × 2
- Account for alignment (typically 32-byte aligned width)

**Reference**: See [references/vb.md](references/vb.md) for pool configuration, buffer calculation, and optimization.

### 9. Region Management (REGION/RGN) Operations

**When to consult**: OSD (timestamps, labels), privacy masking, bounding boxes, graphics overlay

**REGION Types**:
- **OVERLAY** - Bitmap graphics with transparency (ARGB formats)
- **COVER** - Solid privacy masks
- **LINE** - Detection boxes, tracking
- **MOSAIC** - Privacy blur

**Key Operations**:
- Create OVERLAY regions (bitmap graphics with transparency)
- Create COVER regions (solid privacy masks)
- Create LINE regions (detection boxes, tracking)
- Create MOSAIC regions (privacy blur)
- Update OSD content dynamically

**Quick Start (Timestamp OSD)**:
```
1. CVI_RGN_Create() - Create region
2. CVI_RGN_SetBitMap() - Set text/graphics bitmap
3. CVI_RGN_AttachToChn() - Attach to VI/VPSS/VENC
4. CVI_RGN_UpdateCanvas() - Update dynamically
```

**Attachment Targets**:
- Attach to **VI**: OSD on all outputs
- Attach to **VPSS**: OSD on specific resolution stream
- Attach to **VENC**: OSD only in encoded bitstream

**Reference**: See [references/rgn.md](references/rgn.md) for region types, pixel formats, and dynamic updates.

### 10. Geometric Distortion Correction (GDC) Operations

**When to consult**: Lens distortion correction, fisheye dewarp, arbitrary rotation, perspective correction

**Key Operations**:
- LDC (Lens Distortion Correction) for barrel/pincushion
- Fisheye unwarp (convert fisheye to rectilinear)
- Arbitrary angle rotation (not limited to 90°)
- Custom mesh transformation

**Quick Start (LDC)**:
```
1. CVI_GDC_BeginJob() - Create job
2. CVI_GDC_AddLDCTask() - Add correction task
3. CVI_GDC_EndJob() - Execute and wait
```

**Use Cases**:
- Wide-angle surveillance cameras (barrel correction)
- 360° fisheye cameras (dewarp to quad view)
- Document scanning (perspective correction)
- Rotated camera mounting (arbitrary rotation)

**Reference**: See [references/gdc.md](references/gdc.md) for LDC parameters, fisheye modes, and mesh generation.

### 11. Debugging and Troubleshooting

**When to consult**: No video output, frame drops, memory errors, performance issues, ERR_VPSS_NOBUF

**Key Tools**:
- `/proc/cvitek/*` - Module runtime status (vi, vpss, venc, vo, vb, rgn, gdc, sys)
- `/proc/cvitek/log` - Log level control
- `dmesg` - Kernel driver logs
- `CVI_*_QueryStatus()` - Module status APIs

**Quick Diagnostics**:
```bash
# Check binding status (FIRST thing to check for NOBUF errors)
cat /proc/cvitek/sys | grep -A 10 "BIND RELATION"

# Check VI is outputting frames
cat /proc/cvitek/vi

# Check VPSS is receiving/sending
cat /proc/cvitek/vpss

# Check VB buffer availability
cat /proc/cvitek/vb

# Enable debug logs
echo "VI=7" > /proc/cvitek/log
echo "VPSS=7" > /proc/cvitek/log
```

**Common Error: ERR_VPSS_NOBUF (0xc006800e)**:
1. Check binding table (empty = bind failed)
2. Check VPSS RecvCnt (0 = not receiving from VI)
3. Check VB Free buffers (0 = pool exhausted)
4. Ensure correct init order: EnableChn → StartGrp → Bind

**Reference**: See [references/troubleshooting.md](references/troubleshooting.md) for complete diagnostic procedures, error code reference, and recovery steps.

## Common Tasks

### Building a Video Surveillance Camera

**Requirement**: Capture video → Encode H.265 → Stream to network

**Steps**:
1. Consult [references/scenarios.md](references/scenarios.md) → Scenario 1
2. Follow API workflow for VI → VPSS → VENC pipeline
3. Configure multi-resolution encoding (main + sub streams)
4. Implement bitstream retrieval loop
5. Send bitstream to network (RTSP/RTMP/etc.)

### Implementing AI Vision Processing

**Requirement**: Capture video → TPU inference → Draw results → Encode

**Steps**:
1. Consult [references/scenarios.md](references/scenarios.md) → Scenario 5
2. Use VPSS to generate inference input (e.g., 640x640)
3. Get frame from VPSS using `CVI_VPSS_GetChnFrame()`
4. Run TPU inference (via SSCMA or custom model)
5. Draw bounding boxes on frame
6. Send modified frame to encoder or display

### Adding Local Display to Camera

**Requirement**: Camera preview on LCD screen

**Steps**:
1. Consult [references/vo.md](references/vo.md) for VO setup
2. Configure VO for LCD interface (MIPI DSI or I8080)
3. Setup VI → VPSS → VO pipeline
4. VPSS channel should match LCD resolution
5. Bind VPSS to VO for automatic display

## Reference Documentation

Load these references when working with specific modules:

- **[references/vi.md](references/vi.md)** - Video Input: Complete API list, device/pipe/channel management, capture workflows
- **[references/vpss.md](references/vpss.md)** - Video Processing: Scaling, rotation, cropping, format conversion, performance tips
- **[references/venc.md](references/venc.md)** - Video Encoding: H.264/H.265/JPEG encoding, rate control, GOP configuration
- **[references/vo.md](references/vo.md)** - Video Output: Display setup, layer/channel management, interface configuration
- **[references/sys.md](references/sys.md)** - System Control: Module binding, memory management, system initialization
- **[references/vb.md](references/vb.md)** - Video Buffer Pool: Common/private pools, buffer calculation, memory optimization
- **[references/rgn.md](references/rgn.md)** - Region Management: OSD overlay, privacy masking, LINE/COVER/MOSAIC types
- **[references/gdc.md](references/gdc.md)** - Geometric Distortion Correction: LDC, fisheye dewarp, rotation, mesh transformation
- **[references/debug.md](references/debug.md)** - Debugging Guide: /proc filesystem, log system, troubleshooting checklist
- **[references/troubleshooting.md](references/troubleshooting.md)** - Troubleshooting: Error codes, diagnostic decision trees, initialization sequence, common pitfalls
- **[references/scenarios.md](references/scenarios.md)** - Common Scenarios: 6 real-world application examples with complete pipelines

## Key Principles

### 1. Online vs Offline Mode

**Online Mode (Recommended)**:
- Use `CVI_SYS_Bind()` to connect modules
- Zero-copy data flow (hardware handles frame transfer)
- Lower latency, lower CPU usage
- **Use when**: Standard video pipeline, no custom processing needed

**Offline Mode (Advanced)**:
- Use `GetFrame()` and `SendFrame()` APIs
- Manual frame handling (CPU-involved)
- Higher flexibility for custom processing
- **Use when**: Custom algorithms (AI, watermark, etc.), selective frame processing

### 2. Memory Management

**Automatic (VB Pool)**:
- Media modules (VI/VPSS/VENC) use VB pools automatically
- Recommended for standard pipelines
- No manual allocation needed

**Manual (ION)**:
- Use `CVI_SYS_IonAlloc()` for custom buffers
- Required for custom frame manipulation
- Use non-cached for DMA, cached for CPU processing

### 3. Performance Optimization

- **Minimize VPSS groups**: Reuse groups when possible
- **Use hardware binding**: Avoid manual frame loops
- **Match resolutions**: Align sensor → VPSS → VENC sizes
- **Choose appropriate codecs**: H.265 for better compression, H.264 for compatibility
- **Monitor status**: Use `QueryStatus()` APIs to check frame rates and buffer usage

### 4. Error Handling

All CVI APIs return `CVI_S32`:
- `CVI_SUCCESS` (0): Operation succeeded
- Non-zero: Error code (check SDK documentation)

Always check return values for robust applications.

## SDK Location

**Header files**: `/cvi_mpi/include/`
- `cvi_vi.h`, `cvi_vpss.h`, `cvi_venc.h`, `cvi_vo.h`, `cvi_sys.h`
- `cvi_vb.h`, `cvi_region.h`, `cvi_gdc.h`
- `linux/cvi_comm_*.h` (common definitions and structures)

**Libraries**: `/cvi_mpi/lib/`
- `libsys.so`, `libvpss.so`, `libvi.so`, `libvenc.so`, `libvo.so`
- `libvb.so`, `librgn.so`, `libgdc.so`

**Samples**: `/cvi_mpi/sample/`
- `vio/` - VI/VO examples
- `venc/` - Encoding examples
- `region/` - RGN/OSD examples

## Platform Details

### Supported Pixel Formats (PIXEL_FORMAT_E)

The system supports extensive pixel formats for various use cases:

**RGB/BGR Formats**:
- `PIXEL_FORMAT_RGB_888` / `PIXEL_FORMAT_BGR_888` - 24-bit RGB/BGR
- `PIXEL_FORMAT_RGB_888_PLANAR` / `PIXEL_FORMAT_BGR_888_PLANAR` - Planar RGB/BGR

**ARGB Formats** (with alpha channel):
- `PIXEL_FORMAT_ARGB_1555` - 16-bit ARGB
- `PIXEL_FORMAT_ARGB_4444` - 16-bit ARGB
- `PIXEL_FORMAT_ARGB_8888` - 32-bit ARGB

**Bayer Formats** (Sensor RAW):
- `PIXEL_FORMAT_RGB_BAYER_8BPP` / `10BPP` / `12BPP` / `14BPP` / `16BPP`

**YUV Planar Formats**:
- `PIXEL_FORMAT_YUV_PLANAR_422` - YUV 4:2:2 planar
- `PIXEL_FORMAT_YUV_PLANAR_420` - YUV 4:2:0 planar
- `PIXEL_FORMAT_YUV_PLANAR_444` - YUV 4:4:4 planar
- `PIXEL_FORMAT_YUV_400` - Grayscale only

**YUV Semi-Planar Formats**:
- `PIXEL_FORMAT_NV12` / `PIXEL_FORMAT_NV21` - YUV 4:2:0 semi-planar
- `PIXEL_FORMAT_NV16` / `PIXEL_FORMAT_NV61` - YUV 4:2:2 semi-planar

**YUV Packed Formats**:
- `PIXEL_FORMAT_YUYV` / `PIXEL_FORMAT_UYVY` / `PIXEL_FORMAT_YVYU` / `PIXEL_FORMAT_VYUY` - Packed YUV

**HSV Format**:
- `PIXEL_FORMAT_HSV_888` / `PIXEL_FORMAT_HSV_888_PLANAR`

**Deep Learning Formats** (for TPU):
- `PIXEL_FORMAT_FP32_C1` / `C3_PLANAR` - 32-bit float
- `PIXEL_FORMAT_INT32_C1` / `C3_PLANAR` - 32-bit integer
- `PIXEL_FORMAT_UINT32_C1` / `C3_PLANAR` - 32-bit unsigned
- `PIXEL_FORMAT_BF16_C1` / `C3_PLANAR` - 16-bit float
- `PIXEL_FORMAT_INT16_C1` / `C3_PLANAR` - 16-bit integer
- `PIXEL_FORMAT_UINT16_C1` / `C3_PLANAR` - 16-bit unsigned
- `PIXEL_FORMAT_INT8_C1` / `C3_PLANAR` - 8-bit integer
- `PIXEL_FORMAT_UINT8_C1` / `C3_PLANAR` - 8-bit unsigned

### Alignment Requirements

When processing data from memory, different processor modules have specific alignment requirements.

**Alignment Definition**:
- Alignment is the amount of data read/written per row in image processing (must be row-aligned multiple)
- Example: YUV420 PLANAR format at 720x480
  - Y plane: `ALIGN(720, 32) × 480 = 736 × 480`
  - U/V planes: `ALIGN(360, 32) × 240 = 384 × 240`

**CV181X/CV180X Alignment**:
- VI: Module-specific alignment
- VPSS: Module-specific alignment
- VO: Module-specific alignment

Alignment can be modified via APIs like `CVI_VPSS_SetChnAlign`, but cannot go below hardware limits.

**Calculation Example**:
```c
#define ALIGN(x, a) (((x) + (a) - 1) & ~((a) - 1))

CVI_U32 width = 1920;
CVI_U32 height = 1080;
CVI_U32 aligned_width = ALIGN(width, 32);  // 1920 (already aligned)
CVI_U32 stride = aligned_width;  // Stride for memory allocation
```

### Platform Differences (CV181X vs CV180X)

| Feature | CV181X | CV180X |
|---------|--------|--------|
| **VI Max Resolution** | 5M (2880×1620) @ 30fps | 4M (2560×1440) @ 30fps |
| **HDR/WDR Support** | ✅ Yes | ❌ No |
| **VO Module** | ✅ Supported | ❌ Not supported |
| **VDEC H.264** | ✅ Supported | ❌ Not supported |
| **VDEC JPEG/MJPEG** | ✅ Supported | ✅ Supported |
| **VPSS Channels (single input)** | 4 groups | 3 groups |
| **VPSS Channels (dual input)** | 1+3 groups | 1+2 groups |
| **VPSS ONLINE Mode** | ✅ Supported | ✅ Supported |
| **Temperature Monitoring** | ✅ Supported | ✅ Supported |
| **Dual-OS Communication** | ✅ Supported | ✅ Supported |
| **Audio Modules** | ✅ Full support | ✅ Full support |
| **VI/VPSS/VENC** | ✅ Supported | ✅ Supported |

**Key Takeaways**:
- CV180X is a lower-cost variant without VO, H.264 decoding, or HDR support
- Both platforms support the core video pipeline (VI → VPSS → VENC)
- Choose platform based on feature requirements (display, decoding, HDR)

## Notes

### CRITICAL: Initialization Order

**MMF System Depends on Buffer Pools**

The MMF system's normal operation depends on buffer pools. **Incorrect order causes runtime errors**.

**Correct Initialization Sequence**:
```c
1. CVI_VB_SetConfig(pstVbConfig)  // Set VB configuration
2. CVI_VB_Init()                   // Initialize VB (allocates pools)
3. CVI_SYS_Init()                  // Initialize MMF system
```

**Correct Deinitialization Sequence**:
```c
1. CVI_SYS_Exit()                  // Deinitialize MMF system
2. CVI_VB_Exit()                   // Deinitialize VB (frees pools)
```

**⚠️ WARNING**:
- **MUST** call `CVI_VB_Init()` **before** `CVI_SYS_Init()`
- Failure to follow this order will cause abnormal operation
- This is a hard requirement, not a guideline

### Module-Specific Notes

- **VPSS order**: `CreateGrp` → `ResetGrp` → `SetChnAttr` → `EnableChn` → `StartGrp` → `Bind`
- **Binding timing**: Must be AFTER both source and destination modules are started
- Always check VB pool configuration before system init
- Configure modules BEFORE binding
- Unbind modules BEFORE destroying
- Release frames after GetFrame operations
- Always check return values for error handling
- Use `/proc/cvitek/*` to monitor runtime status
- **Binding verification**: Check `/proc/cvitek/sys` for binding table
- Maximum VI channels: 4 per pipe
- Maximum VPSS channels: 4 per group
- Maximum RGN layers: Platform-dependent (typically 4-8)
- Check hardware capabilities for resolution/framerate limits
