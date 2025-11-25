# AI Agent Instructions for HPC Systems

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Supercharge AI coding assistants with HPC-specific knowledge.**

This repository contains comprehensive instruction documents that help AI coding assistants (GitHub Copilot, Claude, ChatGPT, Cursor, etc.) understand and work effectively with High-Performance Computing (HPC) environments.

## 🎯 Purpose

AI coding assistants often struggle with HPC systems because they lack context about:
- Job schedulers (PBS, Slurm)
- Module systems (Lmod, Environment Modules)
- Containerized vs traditional HPC architectures
- Storage hierarchies and quotas
- Platform-specific conventions and best practices

These instruction documents bridge that knowledge gap, enabling AI assistants to:
- ✅ Generate correct job submission scripts
- ✅ Understand storage persistence and quotas
- ✅ Use proper module loading commands
- ✅ Follow platform-specific best practices
- ✅ Avoid common pitfalls that waste compute time

## 📚 Available Instructions

| Platform | Location | Description |
|----------|----------|-------------|
| **UCloud (SDU)** | [`docs/ucloud-sdu.md`](docs/ucloud-sdu.md) | SDU eScience Center's containerized HPC platform (k3s, WekaFS) |
| **Computerome (DTU)** | [`docs/computerome-dtu.md`](docs/computerome-dtu.md) | DTU's traditional HPC cluster (PBS/Torque, Moab) |

## 🚀 Quick Start

### For GitHub Copilot (VS Code)

1. Copy the relevant instruction file to your project:
   ```bash
   # For UCloud
   curl -o .github/copilot-instructions.md https://raw.githubusercontent.com/tuhulab/ai-agent-for-HPC/main/docs/ucloud-sdu.md
   
   # For Computerome
   curl -o .github/copilot-instructions.md https://raw.githubusercontent.com/tuhulab/ai-agent-for-HPC/main/docs/computerome-dtu.md
   ```

2. GitHub Copilot will automatically use this context when assisting you.

### For Claude / ChatGPT / Other AI Assistants

1. Copy the content of the relevant instruction file
2. Paste it at the beginning of your conversation, or
3. Reference the raw GitHub URL in your prompt

### For Cursor

1. Add the instruction file to your `.cursorrules` or workspace settings
2. Or include it in your project's documentation folder

## 📖 Document Structure

Each instruction document follows a consistent format:

```
├── Executive Summary / System Overview
├── Critical Pre-Execution Checks (what to verify first)
├── What NOT to Do (common mistakes)
├── System Architecture
├── Storage Architecture
├── Job Submission Guide
├── Module System
├── Best Practices
├── Troubleshooting Guide
└── Quick Reference
```

## 🔧 Customization

Feel free to fork and customize these documents for your specific:
- Project requirements
- Team conventions
- Additional HPC systems
- Local policies and quotas

## 🤝 Contributing

Contributions are welcome! If you have experience with other HPC systems, please consider:

1. **Adding a new platform**: Create a new file in `docs/` following the existing format
2. **Improving existing docs**: Submit PRs with corrections or additions
3. **Sharing your experience**: Open issues with suggestions or common pitfalls

### Contribution Guidelines

- Follow the existing document structure for consistency
- Include practical examples and code snippets
- Add troubleshooting sections for common issues
- Include disclaimers for unofficial documentation

## ⚠️ Disclaimer

These documents are provided in a personal capacity and are not affiliated with, endorsed by, or representative of any HPC center, university, or organization mentioned. The content is provided "AS IS" without warranties. Always verify information against official documentation and consult your HPC support team for authoritative guidance.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👤 Author

**Tu Hu** - [GitHub](https://github.com/tuhulab) | [LinkedIn](https://www.linkedin.com/in/tuhu/)

---

*If you find this useful, consider giving it a ⭐ on GitHub!*
