# VPSS (Video Processing Subsystem) Module Reference

## Overview

VPSS provides hardware-accelerated video post-processing including:
- Scaling (upscale/downscale)
- Cropping
- Rotation (0/90/180/270)
- Format conversion (YUV420/YUV422/RGB)
- Image enhancement (brightness, contrast, saturation, hue)
- LDC (Lens Distortion Correction)

## Core Concepts

### Group-Channel Architecture

```
VPSS Group → VPSS Channel (0-3)
     |            |
   Input      Scaled Output
  (1 source)  (Multi-resolution)
```

- **Group (Grp)**: Input processing unit (one input source)
- **Channel (Chn)**: Output stream with independent resolution/format

### Typical Data Flow

```
VI/User → VPSS Grp → VPSS Chn → VENC/VO/User
                        ├─ Chn0 (1080p)
                        ├─ Chn1 (720p)
                        └─ Chn2 (360p)
```

## Essential APIs

### Group Management

- `CVI_VPSS_CreateGrp()` - Create VPSS group
- `CVI_VPSS_DestroyGrp()` - Destroy group
- `CVI_VPSS_StartGrp()` - Start group processing
- `CVI_VPSS_StopGrp()` - Stop group processing
- `CVI_VPSS_ResetGrp()` - Reset group (clear buffers)
- `CVI_VPSS_SetGrpAttr()` - Set group attributes (input size, pixel format)
- `CVI_VPSS_GetGrpAttr()` - Get group attributes

### Channel Management

- `CVI_VPSS_SetChnAttr()` - Configure channel (output size, format)
- `CVI_VPSS_GetChnAttr()` - Get channel attributes
- `CVI_VPSS_EnableChn()` - Enable channel output
- `CVI_VPSS_DisableChn()` - Disable channel output
- `CVI_VPSS_ShowChn()` - Resume channel output
- `CVI_VPSS_HideChn()` - Pause channel output

### Image Processing

- `CVI_VPSS_SetGrpCrop()` - Set group-level crop
- `CVI_VPSS_GetGrpCrop()` - Get group crop settings
- `CVI_VPSS_SetChnCrop()` - Set channel-level crop
- `CVI_VPSS_GetChnCrop()` - Get channel crop settings
- `CVI_VPSS_SetChnRotation()` - Set rotation (0/90/180/270)
- `CVI_VPSS_GetChnRotation()` - Get rotation setting
- `CVI_VPSS_SetChnLDCAttr()` - Set lens distortion correction
- `CVI_VPSS_GetChnLDCAttr()` - Get LDC attributes

### Image Enhancement

- `CVI_VPSS_SetGrpProcAmp()` - Set brightness/contrast/saturation/hue
- `CVI_VPSS_GetGrpProcAmp()` - Get enhancement settings

### Frame Operations

- `CVI_VPSS_SendFrame()` - Send frame to group (manual input)
- `CVI_VPSS_GetChnFrame()` - Get processed frame from channel
- `CVI_VPSS_ReleaseChnFrame()` - Release frame buffer
- `CVI_VPSS_SendChnFrame()` - Send frame directly to channel (bypass group)

### Memory Management

- `CVI_VPSS_AttachVbPool()` - Attach video buffer pool
- `CVI_VPSS_DetachVbPool()` - Detach buffer pool

### Advanced Features

- `CVI_VPSS_SetChnScaleCoefLevel()` - Set scaling quality (0-3)
- `CVI_VPSS_SetChnYRatio()` - Set Y/C ratio for format conversion
- `CVI_VPSS_GetRegionLuma()` - Calculate luma statistics for region

## Common Workflows

### Online Mode (Auto-bind from VI)

1. Create group: `CVI_VPSS_CreateGrp()`
2. Configure group: `CVI_VPSS_SetGrpAttr()` (input size from VI)
3. Configure channels: `CVI_VPSS_SetChnAttr()` (output sizes)
4. Start group: `CVI_VPSS_StartGrp()`
5. Enable channels: `CVI_VPSS_EnableChn()`
6. Bind from VI: `CVI_SYS_Bind(VI, VPSS)`
7. Bind to VENC/VO: `CVI_SYS_Bind(VPSS, VENC/VO)`

### Offline Mode (Manual frame input)

1. Setup VPSS as above (steps 1-5)
2. Send frame: `CVI_VPSS_SendFrame()`
3. Get result: `CVI_VPSS_GetChnFrame()`
4. Process frame data
5. Release frame: `CVI_VPSS_ReleaseChnFrame()`

### Cleanup

1. Unbind connections: `CVI_SYS_UnBind()`
2. Disable channels: `CVI_VPSS_DisableChn()`
3. Stop group: `CVI_VPSS_StopGrp()`
4. Destroy group: `CVI_VPSS_DestroyGrp()`

## Key Structures

- `VPSS_GRP_ATTR_S` - Group attributes (input size, format)
- `VPSS_CHN_ATTR_S` - Channel attributes (output size, format, scaling quality)
- `VIDEO_FRAME_INFO_S` - Frame data
- `CROP_INFO_S` - Crop region
- `ROTATION_E` - Rotation enumeration
- `VPSS_LDC_ATTR_S` - LDC attributes
- `VPSS_PROC_AMP_S` - Image enhancement parameters

## Performance Considerations

### Scaling Limits

- **Upscale**: Maximum 16x
- **Downscale**: Maximum 1/32
- Best quality at 1:1 ratio
- Use `SetChnScaleCoefLevel()` for quality vs performance trade-off

### Channel Limitations

- Maximum 4 channels per group (Chn 0-3)
- Channel 0: Highest resolution (up to input size)
- Channel 1-3: Downscaled outputs

### Memory Optimization

- Disable unused channels to save memory
- Use appropriate pixel formats (YUV420 uses less memory than YUV422)
- Attach shared VB pools when possible

## Header Files

- `/cvi_mpi/include/cvi_vpss.h` - Main VPSS API
- `/cvi_mpi/include/linux/cvi_comm_vpss.h` - VPSS common definitions

## Related Modules

- **VI**: Video input source
- **VENC**: Video encoding destination
- **VO**: Video output destination
- **SYS**: System binding
- **VB**: Video buffer management

## Notes

- VPSS operates in **online mode** (auto-bind) or **offline mode** (manual SendFrame)
- Group-level crop applies before channel processing
- Channel-level crop applies after scaling
- Rotation and LDC have performance impact - use only when needed
- Multiple groups can run concurrently (hardware-dependent limits)
