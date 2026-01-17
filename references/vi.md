# VI (Video Input) Module Reference

## Overview

VI module handles video input from camera sensors through the ISP (Image Signal Processor) pipeline. It manages:
- Device configuration (sensor interface)
- Pipe configuration (ISP processing pipeline)
- Channel configuration (output streams)

## Core Concepts

### Three-Layer Architecture

```
VI Device → VI Pipe → VI Channel
   |           |           |
Sensor     ISP Pipeline  Output
Interface   Processing   Streams
```

- **Device (Dev)**: Physical sensor interface (MIPI, DVP, etc.)
- **Pipe**: ISP processing pipeline (RAW → YUV conversion)
- **Channel (Chn)**: Output channels with different resolutions/formats

### Typical Data Flow

```
Sensor → VI Dev → VI Pipe → VI Chn → VPSS/VENC/User
```

## Essential APIs

### Device Management

- `CVI_VI_SetDevAttr()` - Configure device attributes (interface type, data format)
- `CVI_VI_EnableDev()` - Enable video input device
- `CVI_VI_DisableDev()` - Disable video input device
- `CVI_VI_SetDevBindPipe()` - Bind device to pipe
- `CVI_VI_GetDevBindPipe()` - Query device-pipe binding

### Pipe Management

- `CVI_VI_CreatePipe()` - Create ISP processing pipe
- `CVI_VI_DestroyPipe()` - Destroy pipe
- `CVI_VI_SetPipeAttr()` - Configure pipe attributes (resolution, frame rate)
- `CVI_VI_StartPipe()` - Start pipe processing
- `CVI_VI_StopPipe()` - Stop pipe processing
- `CVI_VI_GetPipeFrame()` - Get RAW frame from pipe (for offline ISP)
- `CVI_VI_ReleasePipeFrame()` - Release RAW frame

### Channel Management

- `CVI_VI_SetChnAttr()` - Configure channel attributes (resolution, pixel format)
- `CVI_VI_EnableChn()` - Enable channel output
- `CVI_VI_DisableChn()` - Disable channel output
- `CVI_VI_GetChnFrame()` - Get processed frame from channel
- `CVI_VI_ReleaseChnFrame()` - Release frame buffer

### Image Processing

- `CVI_VI_SetChnCrop()` - Set channel crop region
- `CVI_VI_GetChnCrop()` - Get channel crop settings
- `CVI_VI_SetChnRotation()` - Set rotation (0/90/180/270)
- `CVI_VI_GetChnRotation()` - Get rotation setting
- `CVI_VI_SetChnFlipMirror()` - Set flip/mirror mode
- `CVI_VI_GetChnFlipMirror()` - Get flip/mirror setting
- `CVI_VI_SetChnLDCAttr()` - Set lens distortion correction
- `CVI_VI_GetChnLDCAttr()` - Get LDC attributes

### Memory Management

- `CVI_VI_AttachVbPool()` - Attach video buffer pool to channel
- `CVI_VI_DetachVbPool()` - Detach buffer pool

### Status Query

- `CVI_VI_QueryDevStatus()` - Query device status
- `CVI_VI_QueryPipeStatus()` - Query pipe status
- `CVI_VI_QueryChnStatus()` - Query channel status (frame count, lost frames, etc.)

## Common Workflows

### Basic Video Capture (Online Mode)

1. Initialize system: `CVI_SYS_Init()`
2. Configure and enable device: `CVI_VI_SetDevAttr()` → `CVI_VI_EnableDev()`
3. Bind device to pipe: `CVI_VI_SetDevBindPipe()`
4. Create and configure pipe: `CVI_VI_CreatePipe()` → `CVI_VI_SetPipeAttr()`
5. Start pipe: `CVI_VI_StartPipe()`
6. Configure and enable channel: `CVI_VI_SetChnAttr()` → `CVI_VI_EnableChn()`
7. Bind to next module (VPSS/VENC): `CVI_SYS_Bind()`

### Manual Frame Capture

1. Setup VI as above (steps 1-6)
2. Get frame: `CVI_VI_GetChnFrame()`
3. Process frame data
4. Release frame: `CVI_VI_ReleaseChnFrame()`

### Cleanup

1. Unbind: `CVI_SYS_UnBind()`
2. Disable channel: `CVI_VI_DisableChn()`
3. Stop pipe: `CVI_VI_StopPipe()`
4. Destroy pipe: `CVI_VI_DestroyPipe()`
5. Disable device: `CVI_VI_DisableDev()`

## Key Structures

- `VI_DEV_ATTR_S` - Device attributes
- `VI_PIPE_ATTR_S` - Pipe attributes (resolution, frame rate, pixel format)
- `VI_CHN_ATTR_S` - Channel attributes
- `VIDEO_FRAME_INFO_S` - Frame data (used in Get/Release)
- `CROP_INFO_S` - Crop region
- `ROTATION_E` - Rotation enumeration
- `VI_LDC_ATTR_S` - Lens distortion correction attributes

## Header Files

- `/cvi_mpi/include/cvi_vi.h` - Main VI API
- `/cvi_mpi/include/linux/cvi_comm_vi.h` - VI common definitions
- `/cvi_mpi/include/linux/cvi_comm_video.h` - Video frame structures

## Related Modules

- **ISP**: Low-level image tuning (AE, AWB, etc.)
- **VPSS**: Post-processing (scaling, rotation, etc.)
- **SYS**: System binding and buffer management
- **VB**: Video buffer pool allocation

## Notes

- VI channels can operate in **online mode** (auto-bind to VPSS/VENC) or **offline mode** (manual frame fetch)
- Maximum 4 VI channels per pipe on CV181X/CV182X
- Channel 0 is typically the main stream (highest resolution)
- RAW format requires offline ISP processing
- YUV format can be directly encoded or displayed
