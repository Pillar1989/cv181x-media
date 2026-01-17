# CV181X/CV182X Multimedia API Skill

Expert guidance skill for CV181X/CV182X multimedia development on Sophgo platforms.

## Overview

This skill provides comprehensive knowledge of:
- VI (Video Input)
- VPSS (Video Processing)
- VENC (Video Encoding)
- VO (Video Output)
- SYS (System Control)
- VB (Video Buffer Pool)
- RGN (Region/OSD)
- GDC (Geometric Distortion Correction)
- Debugging and troubleshooting

## Version

Current Version: **v1.0.0**

## Structure

```
cv181x-media/
├── SKILL.md              # Main skill definition
├── references/           # Module reference documentation
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
├── scripts/              # Automation scripts
│   ├── update_from_sdk.sh
│   ├── learn_from_usage.py
│   └── validate_skill.sh
└── .skillrc              # Skill configuration
```

## Self-Learning and Auto-Update

This skill is designed to continuously improve through:

### 1. SDK Updates
Automatically extract and integrate new API information from SDK releases:
```bash
./scripts/update_from_sdk.sh /path/to/new/sdk
```

### 2. Usage Learning
Learn from actual usage patterns and collect feedback:
```bash
./scripts/learn_from_usage.py --feedback-file usage_log.txt
```

### 3. Version Control
All changes are tracked via git:
```bash
git log --oneline  # View change history
git diff v1.0.0    # Compare with previous version
```

## Update Workflow

1. **Detect SDK Update**: Monitor SDK release notes
2. **Extract Changes**: Parse new headers and documentation
3. **Update References**: Regenerate affected reference docs
4. **Validate**: Run validation checks
5. **Commit**: Version control with semantic versioning
6. **Package**: Create new .skill file

## Usage

Install this skill in Claude Code:
```bash
# From .skill file
claude skills install cv181x-media.skill

# Or from git repository
claude skills install git+https://your-repo/cv181x-media.git
```

## Contributing

To improve this skill:

1. Fork the repository
2. Create feature branch
3. Make improvements
4. Submit pull request

### Feedback Collection

When using this skill, report issues or improvements:
- API changes discovered
- Missing information
- Incorrect documentation
- New use cases

## Maintenance

### Regular Tasks

- [ ] Monthly: Check for SDK updates
- [ ] Quarterly: Review usage patterns and update common scenarios
- [ ] Annually: Major version release with comprehensive review

### Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## License

This skill is maintained for internal development use.

## Authors

- Initial creation: 2026-01-17
- Auto-update system: Enabled
