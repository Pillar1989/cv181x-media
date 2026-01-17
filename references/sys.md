# SYS (System Control) Module Reference

## Overview

SYS module provides system-level control for media processing:
- **Module Binding** - Connect modules for automatic data flow
- **Memory Management** - ION memory allocation for zero-copy
- **System Configuration** - VI/VPSS working modes
- **DMA Operations** - Fast memory copy

## Essential APIs

### System Initialization

- `CVI_SYS_Init()` - Initialize system (must call first)
- `CVI_SYS_Exit()` - Cleanup system (call before exit)
- `CVI_SYS_GetVersion()` - Get SDK version
- `CVI_SYS_GetChipId()` - Get chip ID (CV1800/CV1810/CV1811/CV1812/CV1813, etc.)

### Module Binding

Binding enables zero-copy data flow between modules without manual frame transfer.

- `CVI_SYS_Bind()` - Bind source module to destination module
- `CVI_SYS_UnBind()` - Unbind modules
- `CVI_SYS_GetBindbyDest()` - Query source bound to destination
- `CVI_SYS_GetBindbySrc()` - Query destination bound to source

**Supported Binding Paths**:
```
VI → VPSS → VENC → Network/File
VI → VPSS → VO → Display
VI → VENC (direct encoding without scaling)
VI → VO (direct display)
VPSS → VENC (offline encoding)
VPSS → VO (offline display)
```

### Memory Management (ION)

- `CVI_SYS_IonAlloc()` - Allocate ION memory (non-cached)
- `CVI_SYS_IonAlloc_Cached()` - Allocate cached ION memory
- `CVI_SYS_IonFree()` - Free ION memory
- `CVI_SYS_Mmap()` - Map physical address to virtual address (non-cached)
- `CVI_SYS_MmapCache()` - Map physical address (cached)
- `CVI_SYS_Munmap()` - Unmap memory
- `CVI_SYS_IonFlushCache()` - Flush cache to memory
- `CVI_SYS_IonInvalidateCache()` - Invalidate cache from memory

### DMA Operations

- `CVI_SYS_TDMACopy()` - 1D DMA copy
- `CVI_SYS_TDMACopy2D()` - 2D DMA copy (with stride)

### VI/VPSS Mode Configuration

- `CVI_SYS_SetVIVPSSMode()` - Set VI-VPSS working mode
  - **Online mode**: VI directly feeds VPSS (low latency, limited crop/rotate)
  - **Offline mode**: VI outputs to memory, VPSS reads (higher latency, full features)
- `CVI_SYS_GetVIVPSSMode()` - Get VI-VPSS mode
- `CVI_SYS_SetVPSSMode()` - Set VPSS working mode
- `CVI_SYS_GetVPSSMode()` - Get VPSS mode

## Common Workflows

### System Initialization Pattern

```c
// Initialize system
CVI_SYS_Init();

// Setup modules (VI, VPSS, VENC, etc.)
// ...

// Bind modules
MMF_CHN_S stSrcChn = {.enModId = CVI_ID_VI, .s32DevId = 0, .s32ChnId = 0};
MMF_CHN_S stDestChn = {.enModId = CVI_ID_VPSS, .s32DevId = 0, .s32ChnId = 0};
CVI_SYS_Bind(&stSrcChn, &stDestChn);

// Processing happens automatically via binding
// ...

// Cleanup
CVI_SYS_UnBind(&stSrcChn, &stDestChn);
CVI_SYS_Exit();
```

### ION Memory Pattern

```c
// Allocate ION memory
CVI_U64 paddr;
void *vaddr = CVI_SYS_IonAlloc(&paddr, size);

// Use memory for DMA/hardware operations
// ...

// Free ION memory
CVI_SYS_IonFree(paddr, vaddr);
```

### Module Binding Examples

**Example 1: VI → VPSS → VENC (Video encoding pipeline)**
```c
MMF_CHN_S vi_chn = {CVI_ID_VI, 0, 0};
MMF_CHN_S vpss_chn = {CVI_ID_VPSS, 0, 0};
MMF_CHN_S venc_chn = {CVI_ID_VENC, 0, 0};

CVI_SYS_Bind(&vi_chn, &vpss_chn);     // VI feeds VPSS
CVI_SYS_Bind(&vpss_chn, &venc_chn);   // VPSS feeds VENC
```

**Example 2: VI → VO (Direct display)**
```c
MMF_CHN_S vi_chn = {CVI_ID_VI, 0, 0};
MMF_CHN_S vo_chn = {CVI_ID_VO, 0, 0};

CVI_SYS_Bind(&vi_chn, &vo_chn);       // VI directly to VO
```

**Example 3: Multi-channel encoding**
```c
MMF_CHN_S vpss_chn0 = {CVI_ID_VPSS, 0, 0};  // 1080p
MMF_CHN_S vpss_chn1 = {CVI_ID_VPSS, 0, 1};  // 720p
MMF_CHN_S venc_chn0 = {CVI_ID_VENC, 0, 0};
MMF_CHN_S venc_chn1 = {CVI_ID_VENC, 0, 1};

CVI_SYS_Bind(&vpss_chn0, &venc_chn0);
CVI_SYS_Bind(&vpss_chn1, &venc_chn1);
```

## Key Structures

- `MMF_CHN_S` - Module channel identifier
  - `enModId` - Module ID (CVI_ID_VI, CVI_ID_VPSS, CVI_ID_VENC, CVI_ID_VO)
  - `s32DevId` - Device ID
  - `s32ChnId` - Channel ID
- `VI_VPSS_MODE_E` - VI-VPSS working mode enumeration
- `VPSS_MODE_E` - VPSS working mode enumeration

## VI/VPSS Working Modes

### Online Mode
- VI hardware directly feeds VPSS
- **Advantages**: Low latency, zero memory copy
- **Limitations**: Limited crop/rotation features
- **Use case**: Real-time preview, low-latency streaming

### Offline Mode
- VI outputs to memory, VPSS reads from memory
- **Advantages**: Full crop/rotation support, flexible timing
- **Limitations**: Higher latency, memory bandwidth usage
- **Use case**: Complex processing, offline encoding

## Memory Management Notes

### ION vs VB (Video Buffer)
- **ION**: General-purpose physical memory allocation (for custom buffers)
- **VB**: Video buffer pool (for automatic frame management in VI/VPSS/VENC)

### Cache Considerations
- Use **non-cached** for hardware DMA (faster hardware access)
- Use **cached** for CPU-intensive operations (faster CPU access)
- Always flush cache after CPU writes to cached memory
- Always invalidate cache before CPU reads from cached memory

## Header Files

- `/cvi_mpi/include/cvi_sys.h` - Main SYS API
- `/cvi_mpi/include/linux/cvi_comm_sys.h` - SYS common definitions
- `/cvi_mpi/include/linux/cvi_defines.h` - Module ID definitions

## Related Modules

- **VB**: Video buffer pool (automatic memory management for media modules)
- **VI/VPSS/VENC/VO**: Media processing modules

## Notes

- Always call `CVI_SYS_Init()` before any media module operations
- Always call `CVI_SYS_Exit()` when shutting down
- Binding must be established AFTER modules are configured and started
- Unbinding must be done BEFORE modules are stopped
- ION memory is for custom usage; most media modules use VB pools automatically
- Use `CVI_SYS_TDMACopy()` for fast memory-to-memory copy instead of memcpy
