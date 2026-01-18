# Concurrent Scenarios Reference

## Overview

This document provides complete implementation examples for running multiple multimedia scenarios concurrently on CV181X/CV182X platforms. Multi-scenario designs are common in applications like:

- **Smart Cameras**: Live camera preview + file playback + AI inference
- **Video Conference**: Camera encoding + remote decoding + display
- **Surveillance Systems**: Multiple camera recording + motion detection
- **AI Vision Systems**: Camera input + TPU inference + result encoding

**Key Principle**: Use **separate VPSS Groups** for different input sources. VPSS Groups cannot dynamically switch between Bind mode (VI) and SendFrame mode (file/manual input).

## Architecture

```
┌─────────────┐
│   Camera    │
│  (VI Pipe)  │
└──────┬──────┘
       │
       ├───────► ┌──────────────┐
       │         │ VPSS Group 0 │
       │         │ (Bind Mode)  │
       │         └───────┬──────┘
       │                 │
       │         ┌───────▼───────┐
       │         │  VPSS Chn 0   │ ──► Save/Display
       │         │  (1080p)      │
       │         └───────────────┘
       │
       │         ┌───────────────┐
       │         │  VPSS Chn 1   │ ──► Save/Display
       │         │  (720p)       │
       │         └───────────────┘
       │
┌──────▼──────┐
│   File      │
│  (Memory)   │
└──────┬──────┘
       │
       │         ┌──────────────┐
       └────────►│ VPSS Group 1 │
       │         │ (SendFrame)  │
       │         └───────┬──────┘
       │                 │
       │         ┌───────▼───────┐
       │         │  VPSS Chn 0   │ ──► VENC
       │         │  (1080p)      │
       │         └───────────────┘
       │
       │         ┌───────────────┐
       │         │  VPSS Chn 1   │ ──► TPU Inference
       │         │  (640x640)    │      │
       │         └───────────────┘      ▼
       │                                  Draw → Save
       └────────► ┌──────────────┐
                  │ VENC Chn 0    │ ──► Save Bitstream
                  │ (H.264/H.265) │
                  └───────────────┘
```

## Scenario 1: Camera → VPSS → Save/Display (Online Mode)

**Use Case**: Live camera preview with multiple resolution outputs.

### Configuration

```c
// VB Configuration
VB_CONFIG_S vb_config = {
    .u32MaxPoolCnt = 1,
    .astCommPool[0] = {
        .u32BlkSize = 1920 * 1080 * 3 / 2,  // YUV420
        .u32BlkCnt = 8,
        .acName = "Camera_VPSS"
    }
};
CVI_VB_SetConfig(&vb_config);
CVI_VB_Init();
CVI_SYS_Init();

// VI Device (Camera)
VI_DEV_ATTR_S dev_attr = {
    .enIntfMode = VI_INTF_MODE_MIPI,
    .enWorkMode = VI_WORK_MODE_1Multiplex,
    // ... sensor-specific settings
};
CVI_VI_SetDevAttr(0, &dev_attr);
CVI_VI_EnableDev(0);

// VI Pipe
VI_PIPE_ATTR_S pipe_attr = {
    .enMastPipeMode = VI_ONLINE_VPSS_OFFLINE,
    .u32MaxW = 1920,
    .u32MaxH = 1080,
    .enPixFmt = PIXEL_FORMAT_YUV_PLANAR_420,
};
CVI_VI_CreatePipe(0, &pipe_attr);
CVI_VI_SetPipeAttr(0, &pipe_attr);
CVI_VI_StartPipe(0);

// VI Channel
VI_CHN_ATTR_S chn_attr = {
    .stIspOpt = {...},
    .stFrameRate = {30, 1},
    .enPixFmt = PIXEL_FORMAT_YUV_PLANAR_420,
    .u32Width = 1920,
    .u32Height = 1080,
};
CVI_VI_SetChnAttr(0, 0, &chn_attr);
CVI_VI_EnableChn(0, 0);

// VPSS Group 0 (Bind Mode)
VPSS_GRP_ATTR_S grp_attr = {
    .u32MaxW = 1920,
    .u32MaxH = 1080,
    .enPixFmt = PIXEL_FORMAT_YUV_PLANAR_420,
};
VPSS_GRP grp_camera = 0;
CVI_VPSS_CreateGrp(grp_camera, &grp_attr);
CVI_VPSS_ResetGrp(grp_camera);

// Channel 0: 1080p output
VPSS_CHN_ATTR_S chn_attr_1080p = {
    .u32Width = 1920,
    .u32Height = 1080,
    .enPixFmt = PIXEL_FORMAT_YUV_PLANAR_420,
};
CVI_VPSS_SetChnAttr(grp_camera, 0, &chn_attr_1080p);
CVI_VPSS_EnableChn(grp_camera, 0);

// Channel 1: 720p output
VPSS_CHN_ATTR_S chn_attr_720p = {
    .u32Width = 1280,
    .u32Height = 720,
    .enPixFmt = PIXEL_FORMAT_YUV_PLANAR_420,
};
CVI_VPSS_SetChnAttr(grp_camera, 1, &chn_attr_720p);
CVI_VPSS_EnableChn(grp_camera, 1);

CVI_VPSS_StartGrp(grp_camera);

// Bind VI → VPSS
MMF_CHN_S vi_chn = {CVI_ID_VI, 0, 0};
MMF_CHN_S vpss_grp0 = {CVI_ID_VPSS, grp_camera, 0};
CVI_SYS_Bind(&vi_chn, &vpss_grp0);
```

### Get Frames and Save

```c
while (camera_running) {
    VIDEO_FRAME_INFO_S frame;

    // Get 1080p frame
    CVI_VPSS_GetChnFrame(grp_camera, 0, &frame, -1);
    save_frame_to_file(&frame, "camera_1080p.yuv");
    CVI_VPSS_ReleaseChnFrame(grp_camera, 0, &frame);

    // Get 720p frame
    CVI_VPSS_GetChnFrame(grp_camera, 1, &frame, -1);
    save_frame_to_file(&frame, "camera_720p.yuv");
    CVI_VPSS_ReleaseChnFrame(grp_camera, 1, &frame);
}
```

## Scenario 2: File → VPSS → VENC → Save Bitstream (Offline Mode)

**Use Case**: Encode files to H.264/H.265 bitstreams.

### Configuration

```c
// VPSS Group 1 (SendFrame Mode)
VPSS_GRP grp_file = 1;
CVI_VPSS_CreateGrp(grp_file, &grp_attr);
CVI_VPSS_ResetGrp(grp_file);
CVI_VPSS_SetChnAttr(grp_file, 0, &chn_attr_1080p);
CVI_VPSS_EnableChn(grp_file, 0);
CVI_VPSS_StartGrp(grp_file);
// NO Bind - will use SendFrame

// VENC Channel
VENC_CHN_ATTR_S venc_attr = {
    .stVencAttr = {
        .enType = PT_H265,  // H.265 encoding
        .enPixelFormat = PIXEL_FORMAT_YUV_PLANAR_420,
        .u32MaxWidth = 1920,
        .u32MaxHeight = 1080,
    },
    .stRcAttr = {
        .enRcMode = VENC_RC_MODE_H265CBR,
        .stH265Cbr = {
            .u32BitRate = 4000000,  // 4 Mbps
            .u32Gop = 30,
            .fr32DstFrameRate = {30, 1},
        },
    },
};
VENC_CHN venc_chn = 0;
CVI_VENC_CreateChn(venc_chn, &venc_attr);
CVI_VENC_StartRecvFrame(venc_chn, NULL);

// Bind VPSS → VENC
MMF_CHN_S vpss_grp1 = {CVI_ID_VPSS, grp_file, 0};
MMF_CHN_S venc_chn0 = {CVI_ID_VENC, venc_chn, 0};
CVI_SYS_Bind(&vpss_grp1, &venc_chn0);
```

### Send Frames from File

```c
VB_CAL_CONFIG_S vb_cfg;
COMMON_GetPicBufferConfig(1920, 1080, PIXEL_FORMAT_YUV_PLANAR_420,
                         DATA_BITWIDTH_8, COMPRESS_MODE_NONE,
                         DEFAULT_ALIGN, &vb_cfg);

while (file_running) {
    // 1. Read file to memory
    void *file_data = read_file_to_memory("frame.yuv");

    // 2. Allocate VB block
    VB_BLK blk = CVI_VB_GetBlock(VB_INVALID_POOLID, vb_cfg.u32VBSize);
    if (blk == VB_INVALID_HANDLE) {
        printf("Failed to get VB block\n");
        free(file_data);
        continue;
    }

    // 3. Construct VIDEO_FRAME_INFO_S
    VIDEO_FRAME_INFO_S frame;
    memset(&frame, 0, sizeof(frame));

    frame.stVFrame.u32Width = 1920;
    frame.stVFrame.u32Height = 1080;
    frame.stVFrame.enPixelFormat = PIXEL_FORMAT_YUV_PLANAR_420;
    frame.stVFrame.enCompressMode = COMPRESS_MODE_NONE;

    // Get addresses from VB block
    frame.stVFrame.u64PhyAddr[0] = CVI_VB_Handle2PhysAddr(blk);
    frame.stVFrame.pu8VirAddr[0] = CVI_VB_GetBlockVirAddr(blk);
    frame.stVFrame.u32Stride[0] = vb_cfg.u32MainStride;
    frame.stVFrame.u32Length[0] = vb_cfg.u32MainYSize;

    frame.stVFrame.u64PhyAddr[1] = frame.stVFrame.u64PhyAddr[0] + vb_cfg.u32MainYSize;
    frame.stVFrame.pu8VirAddr[1] = frame.stVFrame.pu8VirAddr[0] + vb_cfg.u32MainYSize;
    frame.stVFrame.u32Stride[1] = vb_cfg.u32CStride;
    frame.stVFrame.u32Length[1] = vb_cfg.u32MainCSize;

    frame.stVFrame.u64PhyAddr[2] = frame.stVFrame.u64PhyAddr[1] + vb_cfg.u32MainCSize;
    frame.stVFrame.pu8VirAddr[2] = frame.stVFrame.pu8VirAddr[1] + vb_cfg.u32MainCSize;
    frame.stVFrame.u32Stride[2] = vb_cfg.u32CStride;
    frame.stVFrame.u32Length[2] = vb_cfg.u32MainCSize;

    // CRITICAL: Set Pool ID
    frame.u32PoolId = CVI_VB_Handle2PoolId(blk);

    // 4. Copy data to VB block
    memcpy(frame.stVFrame.pu8VirAddr[0], file_data, vb_cfg.u32VBSize);

    // 5. Cache invalidation (if using cached ION)
    CVI_SYS_IonInvalidateCache(frame.stVFrame.u64PhyAddr[0],
                               frame.stVFrame.pu8VirAddr[0],
                               frame.stVFrame.u32Length[0]);

    // 6. Send to VPSS (automatically forwarded to VENC via Bind)
    CVI_VPSS_SendFrame(grp_file, &frame, -1);

    // 7. Get encoded bitstream
    VENC_STREAM_S stream;
    CVI_VENC_GetStream(venc_chn, &stream, -1);
    save_bitstream_to_file(&stream, "output.h265");
    CVI_VENC_ReleaseStream(venc_chn, &stream);

    // 8. Cleanup
    free(file_data);
    CVI_VB_ReleaseBlock(blk);
}
```

## Scenario 3: VPSS → TPU Inference → Draw Results → Save

**Use Case**: AI-powered camera with object detection and result visualization.

### Workflow Overview

1. VPSS generates TPU-compatible frame (e.g., 640x640 RGB)
2. Get frame from VPSS using `CVI_VPSS_GetChnFrame()`
3. Prepare TPU input buffer with proper alignment and format
4. Run TPU inference
5. Draw bounding boxes on original frame
6. Save or encode result

### Memory Management for TPU

**Critical Considerations**:
- **Alignment**: TPU input typically requires 4096-byte alignment
- **Format**: TPU may require RGB (not YUV)
- **Buffer Lifetime**: TPU input buffer must remain valid during inference
- **Zero-Copy**: Only possible if VPSS output format matches TPU requirements

### Implementation

```c
// VPSS Configuration for TPU Input
VPSS_GRP grp_tpu = 2;
VPSS_CHN_ATTR_S chn_attr_tpu = {
    .u32Width = 640,
    .u32Height = 640,
    .enPixFmt = PIXEL_FORMAT_RGB_888_PLANAR,  // RGB for TPU
};
CVI_VPSS_SetChnAttr(grp_tpu, 0, &chn_attr_tpu);
CVI_VPSS_EnableChn(grp_tpu, 0);

// TPU Inference Loop
while (tpu_running) {
    VIDEO_FRAME_INFO_S frame;

    // 1. Get frame from VPSS
    CVI_VPSS_GetChnFrame(grp_tpu, 0, &frame, -1);

    // 2. Prepare TPU input buffer
    CVI_U32 tpu_input_size = 640 * 640 * 3;  // RGB, 3 channels
    void *tpu_input = NULL;

    // Option A: Allocate aligned buffer (safe, portable)
    if (posix_memalign(&tpu_input, 4096, tpu_input_size) != 0) {
        printf("Failed to allocate aligned buffer\n");
        CVI_VPSS_ReleaseChnFrame(grp_tpu, 0, &frame);
        continue;
    }

    // Option B: Use VPSS frame directly (zero-copy, requires format match)
    // tpu_input = frame.stVFrame.pu8VirAddr[0];

    // 3. Convert and copy data (if needed)
    if (frame.stVFrame.enPixelFormat != PIXEL_FORMAT_RGB_888_PLANAR) {
        convert_yuv_to_rgb(&frame.stVFrame, tpu_input, 640, 640);
    } else {
        // Format matches - direct copy
        memcpy(tpu_input, frame.stVFrame.pu8VirAddr[0], tpu_input_size);
    }

    // 4. Run TPU inference
    tpu_result_t *result = tpu_inference(tpu_input, 640, 640);
    if (result == NULL) {
        printf("TPU inference failed\n");
        if (tpu_input != frame.stVFrame.pu8VirAddr[0]) {
            free(tpu_input);
        }
        CVI_VPSS_ReleaseChnFrame(grp_tpu, 0, &frame);
        continue;
    }

    // 5. Draw bounding boxes on original frame
    draw_bounding_boxes(&frame, result);

    // 6. Save result or send to encoder
    save_frame_with_bboxes(&frame, result, "tpu_result.yuv");

    // 7. Release frame
    CVI_VPSS_ReleaseChnFrame(grp_tpu, 0, &frame);

    // 8. Free TPU input buffer (if not zero-copy)
    if (tpu_input != frame.stVFrame.pu8VirAddr[0]) {
        free(tpu_input);
    }

    // 9. Free TPU result
    free_tpu_result(result);
}
```

### TPU Input Buffer Options

| Option | Advantages | Disadvantages | Use When |
|--------|------------|---------------|----------|
| **Aligned allocation** | Portable, safe | Extra memcpy | Format conversion needed |
| **Zero-copy** | Best performance | Requires format match | VPSS output = TPU input format |
| **ION buffer** | Shared with hardware | Complex setup | DMA operations needed |

### Format Conversion Example

```c
void convert_yuv_to_rgb(VIDEO_FRAME_S *yuv_frame, void *rgb_buf,
                        CVI_U32 width, CVI_U32 height)
{
    CVI_U8 *y_plane = yuv_frame->pu8VirAddr[0];
    CVI_U8 *u_plane = yuv_frame->pu8VirAddr[1];
    CVI_U8 *v_plane = yuv_frame->pu8VirAddr[2];
    CVI_U32 y_stride = yuv_frame->u32Stride[0];
    CVI_U32 uv_stride = yuv_frame->u32Stride[1];

    CVI_U8 *rgb_out = (CVI_U8 *)rgb_buf;

    for (CVI_U32 y = 0; y < height; y++) {
        for (CVI_U32 x = 0; x < width; x++) {
            CVI_U32 y_idx = y * y_stride + x;
            CVI_U32 uv_idx = (y / 2) * uv_stride + (x / 2);

            CVI_S32 y_val = y_plane[y_idx] - 16;
            CVI_S32 u_val = u_plane[uv_idx] - 128;
            CVI_S32 v_val = v_plane[uv_idx] - 128;

            CVI_S32 r_val = (298 * y_val + 409 * v_val + 128) >> 8;
            CVI_S32 g_val = (298 * y_val - 100 * u_val - 208 * v_val + 128) >> 8;
            CVI_S32 b_val = (298 * y_val + 516 * u_val + 128) >> 8;

            rgb_out[(y * width + x) * 3 + 0] = CLIP(r_val, 0, 255);
            rgb_out[(y * width + x) * 3 + 1] = CLIP(g_val, 0, 255);
            rgb_out[(y * width + x) * 3 + 2] = CLIP(b_val, 0, 255);
        }
    }
}
```

### Drawing Bounding Boxes

```c
void draw_bounding_boxes(VIDEO_FRAME_INFO_S *frame, tpu_result_t *result)
{
    // Draw on Y plane (brightness)
    CVI_U8 *y_plane = frame->stVFrame.pu8VirAddr[0];
    CVI_U32 stride = frame->stVFrame.u32Stride[0];

    for (CVI_U32 i = 0; i < result->count; i++) {
        bbox_t *box = &result->boxes[i];

        // Draw rectangle (simple implementation)
        for (CVI_U32 y = box->y; y < box->y + box->h && y < frame->stVFrame.u32Height; y++) {
            // Top edge
            if (y >= box->y && y < box->y + 2) {
                for (CVI_U32 x = box->x; x < box->x + box->w && x < frame->stVFrame.u32Width; x++) {
                    y_plane[y * stride + x] = 255;  // White
                }
            }
            // Bottom edge
            if (y >= box->y + box->h - 2 && y < box->y + box->h) {
                for (CVI_U32 x = box->x; x < box->x + box->w && x < frame->stVFrame.u32Width; x++) {
                    y_plane[y * stride + x] = 255;  // White
                }
            }
            // Left/right edges
            if (y >= box->y && y < box->y + box->h) {
                if (box->x < frame->stVFrame.u32Width) {
                    y_plane[y * stride + box->x] = 255;
                }
                if (box->x + box->w < frame->stVFrame.u32Width) {
                    y_plane[y * stride + box->x + box->w] = 255;
                }
            }
        }
    }
}
```

## Scenario 4: Complete Concurrent Example

**Use Case**: Camera preview + file encoding + TPU inference running simultaneously.

### VB Pool Configuration (Multi-Scenario)

```c
VB_CONFIG_S vb_config = {
    .u32MaxPoolCnt = 3,

    // Pool 0: Camera → VPSS
    .astCommPool[0] = {
        .u32BlkSize = 1920 * 1080 * 3 / 2,
        .u32BlkCnt = 8,
        .acName = "Camera_VPSS"
    },

    // Pool 1: File → VPSS → VENC
    .astCommPool[1] = {
        .u32BlkSize = 1920 * 1080 * 3 / 2,
        .u32BlkCnt = 4,
        .acName = "File_VPSS_VENC"
    },

    // Pool 2: TPU inference
    .astCommPool[2] = {
        .u32BlkSize = 640 * 640 * 3,  // RGB
        .u32BlkCnt = 4,
        .acName = "TPU_Inference"
    }
};
```

### Complete Pipeline

```c
// Thread 1: Camera → VPSS → Save
void* camera_thread(void* arg) {
    VIDEO_FRAME_INFO_S frame;
    while (running) {
        CVI_VPSS_GetChnFrame(0, 0, &frame, -1);  // 1080p
        save_frame(&frame, "camera_1080p.yuv");
        CVI_VPSS_ReleaseChnFrame(0, 0, &frame);

        CVI_VPSS_GetChnFrame(0, 1, &frame, -1);  // 720p
        save_frame(&frame, "camera_720p.yuv");
        CVI_VPSS_ReleaseChnFrame(0, 1, &frame);
    }
    return NULL;
}

// Thread 2: File → VPSS → VENC
void* file_encode_thread(void* arg) {
    while (running) {
        // Send file frame to VPSS
        VB_BLK blk = CVI_VB_GetBlock(VB_INVALID_POOLID, size);
        // ... load file data ...
        VIDEO_FRAME_INFO_S frame = {...};
        CVI_VPSS_SendFrame(1, &frame, -1);

        // Get encoded stream
        VENC_STREAM_S stream;
        CVI_VENC_GetStream(0, &stream, -1);
        save_stream(&stream, "output.h265");
        CVI_VENC_ReleaseStream(0, &stream);

        CVI_VB_ReleaseBlock(blk);
    }
    return NULL;
}

// Thread 3: VPSS → TPU → Draw → Save
void* tpu_thread(void* arg) {
    VIDEO_FRAME_INFO_S frame;
    while (running) {
        CVI_VPSS_GetChnFrame(2, 0, &frame, -1);

        // TPU inference
        void *tpu_input = prepare_tpu_input(&frame);
        tpu_result_t *result = tpu_inference(tpu_input, 640, 640);

        // Draw results
        draw_bboxes(&frame, result);

        // Save
        save_frame(&frame, "tpu_result.yuv");

        // Cleanup
        CVI_VPSS_ReleaseChnFrame(2, 0, &frame);
        free(tpu_input);
        free_tpu_result(result);
    }
    return NULL;
}

// Main: Start all threads
int main() {
    // Initialize all modules...
    // ...

    // Create threads
    pthread_t camera_tid, file_tid, tpu_tid;
    pthread_create(&camera_tid, NULL, camera_thread, NULL);
    pthread_create(&file_tid, NULL, file_encode_thread, NULL);
    pthread_create(&tpu_tid, NULL, tpu_thread, NULL);

    // Wait for completion
    pthread_join(camera_tid, NULL);
    pthread_join(file_tid, NULL);
    pthread_join(tpu_tid, NULL);

    // Cleanup
    // ...
}
```

## Cleanup

```c
// Unbind all modules
CVI_SYS_UnBind(&vpss_grp1, &venc_chn0);

// Stop VENC
CVI_VENC_StopRecvFrame(venc_chn);
CVI_VENC_DestroyChn(venc_chn);

// Stop VPSS Groups
CVI_VPSS_StopGrp(grp_file);
CVI_VPSS_DestroyGrp(grp_file);

CVI_VPSS_StopGrp(grp_camera);
CVI_VPSS_DestroyGrp(grp_camera);

CVI_VPSS_StopGrp(grp_tpu);
CVI_VPSS_DestroyGrp(grp_tpu);

// Stop VI
CVI_VI_DisableChn(0, 0);
CVI_VI_StopPipe(0);
CVI_VI_DisableDev(0);

// Exit system
CVI_SYS_Exit();
CVI_VB_Exit();
```

## Performance Tips

1. **Use Separate Pools**: Allocate VB pools per scenario to avoid contention
2. **Match Resolutions**: Align VPSS output with downstream requirements
3. **Thread Safety**: Use separate threads for each independent pipeline
4. **Zero-Copy**: Prefer Bind mode over GetFrame/SendFrame when possible
5. **Format Matching**: Configure VPSS output format to match TPU/encoder requirements
6. **Buffer Sizing**: Allocate sufficient buffers in each pool (typical: 4-8 blocks)

## Troubleshooting

### Issue: VB Pool Exhaustion

**Symptom**: `CVI_VB_GetBlock()` returns `VB_INVALID_HANDLE`

**Solution**:
- Increase `.u32BlkCnt` in VB pool configuration
- Release blocks promptly after use
- Use separate pools for different scenarios

### Issue: Frame Drops in Camera Pipeline

**Symptom**: Missing frames when saving from VPSS

**Solution**:
- Check VI is producing frames: `cat /proc/cvitek/vi`
- Verify VPSS is receiving frames: `cat /proc/cvitek/vpss`
- Ensure binding is active: `cat /proc/cvitek/sys | grep BIND`

### Issue: TPU Inference Slow

**Symptom**: TPU inference bottleneck

**Solution**:
- Use zero-copy when format matches
- Allocate TPU input buffer with proper alignment (4096 bytes)
- Reduce VPSS output resolution if possible
- Process every Nth frame (frame skipping)

## Related Modules

- **[VI Module Reference](vi.md)** - Video input setup
- **[VPSS Module Reference](vpss.md)** - Video processing configuration
- **[VENC Module Reference](venc.md)** - Video encoding
- **[VB Module Reference](vb.md)** - Buffer pool management
- **[SYS Module Reference](sys.md)** - System binding and initialization
