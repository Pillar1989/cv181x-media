# Changelog

All notable changes to the CV181X Media Skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-17

### Added
- Initial release of CV181X/CV182X multimedia API skill
- **Core Modules Documentation**:
  - VI (Video Input) - Complete API reference with 60+ APIs
  - VPSS (Video Processing Subsystem) - 50+ APIs for scaling, rotation, cropping
  - VENC (Video Encoding) - 100+ APIs for H.264/H.265/JPEG/MJPEG encoding
  - VO (Video Output) - 30+ APIs for LCD/HDMI display
  - SYS (System Control) - Module binding and memory management
  - VB (Video Buffer Pool) - Unified memory management with 11 APIs
  - RGN (Region Management) - OSD overlay and graphics with 12 APIs
  - GDC (Geometric Distortion Correction) - LDC, fisheye dewarp, rotation with 11 APIs
- **Reference Documentation**:
  - Complete API lists extracted from SDK headers
  - Workflow guides for each module
  - Performance optimization tips
  - Common pitfalls and solutions
- **Debugging Guide** (`debug.md`):
  - /proc filesystem usage for runtime monitoring
  - Log system control
  - Common issue troubleshooting
  - Performance profiling methods
- **Scenarios** (`scenarios.md`):
  - 6 real-world application examples
  - Video surveillance camera
  - Smart doorbell with display
  - Image processing device
  - Video conference device
  - AI-powered security camera
  - Multi-channel NVR
- **Auto-Update System**:
  - `update_from_sdk.sh` - Extract API changes from new SDK releases
  - `learn_from_usage.py` - Analyze usage patterns and collect feedback
  - `validate_skill.sh` - Validate skill integrity
- **Git Version Control**:
  - Repository initialization
  - Semantic versioning support
  - Changelog tracking
- **Configuration**:
  - `.skillrc` - Skill configuration file
  - `.gitignore` - Git ignore rules

### Documentation Structure
```
cv181x-media/
├── SKILL.md              # Main skill (11 capability sections)
├── README.md             # Skill overview and usage
├── CHANGELOG.md          # Version history
├── .skillrc              # Configuration
├── references/           # 10 module reference docs
│   ├── vi.md
│   ├── vpss.md
│   ├── venc.md
│   ├── vo.md
│   ├── sys.md
│   ├── vb.md
│   ├── rgn.md
│   ├── gdc.md
│   ├── debug.md
│   └── scenarios.md
└── scripts/              # Automation scripts
    ├── update_from_sdk.sh
    ├── learn_from_usage.py
    └── validate_skill.sh
```

### Features
- **Comprehensive Coverage**: 11 core capability sections covering all multimedia modules
- **Progressive Disclosure**: Core workflows in SKILL.md, detailed APIs in references
- **Real-World Scenarios**: 6 complete application examples with full pipelines
- **Self-Learning**: Automatic feedback collection and improvement suggestions
- **Version Controlled**: Full git integration for tracking changes
- **Auto-Update**: Detect SDK changes and extract new APIs automatically

### Technical Details
- Total APIs documented: 300+
- Modules covered: 8 (VI, VPSS, VENC, VO, SYS, VB, RGN, GDC)
- Reference documentation: 10 files (~15,000 words)
- Code examples: 30+ workflow examples
- Debug commands: 20+ /proc filesystem checks

### Platform Support
- SG2002 (CV182X)
- SG2000 (CV180X)
- Compatible with reCamera-OS SDK

### Known Limitations
- Manual review required after SDK updates (auto-extraction only)
- Learning system requires user feedback logs
- Some platform-specific features may vary

## [Unreleased]

### Planned
- ISP (Image Signal Processor) module documentation
- VDEC (Video Decoding) module documentation
- Audio module (AIO, AENC, ADEC) integration
- More scenario examples (PTZ camera, object tracking, etc.)
- Interactive troubleshooting flowcharts
- Performance benchmarking tools

---

## Version History

- **v1.0.0** (2026-01-17): Initial release with 8 core modules and auto-update system
