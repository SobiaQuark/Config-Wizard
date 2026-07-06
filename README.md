# Config Wizard

A collection of Claude plugins and skills for QuarksUp / Komeo HR workflows.

## Contents

### Form Builder
Recreates a QuarksUp career assessment form (*formulaire d'entretien carrière*) from a JSON export via the Chrome extension, then verifies it and returns the link.

**Structure:**
```
Form builder/
├── Claude-plugin/
│   └── plugin.json        # Plugin manifest
└── Skills/
    └── assessment-form-builder/
        ├── SKILL.md                        # Skill definition
        └── references/
            ├── competency-config.md        # Competency configuration
            ├── json-mapping.md             # JSON field mapping
            └── type-mapping.md             # Field type mapping
```

## Author
Sobia — QuarksUp
