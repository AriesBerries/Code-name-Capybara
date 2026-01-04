# Code name Capybara

**AI-assisted developer workflow orchestration platform**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-in%20development-yellow.svg)]()

## Overview

Code name Capybara is an Internal Developer Platform (IDP) with AI-native architecture that ensures structural impossibility of desynchronization across code, documentation, and specifications.

**Key Features:**
- 🎯 **Active Workflow Orchestration**: Modular dashboards that execute development tasks
- 🔒 **Single Source of Truth**: Guaranteed propagation across all artifacts
- 🤖 **AI-Native Architecture**: Context-aware assistance throughout development lifecycle
- 🎨 **Extreme Customization**: Every component tailorable without code changes

## Quick Start

1. Read the [Vision Specification](docs/artifacts/ARTIFACT_01_VISION_SPECIFICATION.md)
2. Review [Architecture Decisions](docs/artifacts/ARTIFACT_15_ARCHITECTURE_DECISION_RECORD.md)
3. Follow [Implementation Plan](docs/artifacts/ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md)

## Repository Structure

```
Code-name-Capybara/
├── docs/
│   ├── artifacts/          # Core specifications (22 files)
│   ├── specifications/     # TIER X UI/UX specs
│   ├── guides/            # Tools, workflows, templates
│   ├── adr/               # Architecture Decision Records
│   ├── media/             # UI mockups & diagrams
│   └── archive/           # Historical decision documentation
├── src/                   # Source code (to be developed)
├── infra/                 # Infrastructure (future)
└── scripts/               # Utility scripts (future)
```

## Technology Stack

- **Frontend**: Elm (type-safe UI)
- **Backend**: Go (high-performance API)
- **Validation**: Rust (memory-safe atomic enforcement)
- **Database**: TiDB (distributed SQL)
- **AI**: Anthropic Claude API

## Project Status

**Status**: Specification Complete, Implementation Starting  
**Version**: 1.0.0  
**Timeline**: 3-month MVP (see [Implementation Plan](docs/artifacts/ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md))

## Documentation

All specifications are in `docs/artifacts/`. Start with:

- [ARTIFACT_01_VISION_SPECIFICATION.md](docs/artifacts/ARTIFACT_01_VISION_SPECIFICATION.md)
- [ARTIFACT_02_TERMINOLOGY_GLOSSARY.md](docs/artifacts/ARTIFACT_02_TERMINOLOGY_GLOSSARY.md)
- [ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md](docs/artifacts/ARTIFACT_19_DEVGUIDE_COCKPIT_IMPLEMENTATION_PLAN_V2.md)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Contact

For questions or feedback, please open an issue.
