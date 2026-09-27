<div align="center">

# MDT Windows Image Builder

**Build your own Windows installation image with UUPDump, customize it in audit mode, and capture it for MDT deployment — or any other deployment method.**

[![Windows](https://img.shields.io/badge/Windows-10%20%7C%2011-0078D4)](https://www.microsoft.com/windows)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE.md)
[![Sponsor](https://img.shields.io/badge/Sponsor-ArthurJDurand-ea4aaa?logo=github-sponsors&logoColor=white)](https://github.com/sponsors/ArthurJDurand)

[What This Is](#what-this-is) · [Workflow](#workflow) · [Quick Start](#quick-start) · [Documentation](#documentation) · [Support](#support-this-project) · [Contributing](#contributing)

</div>

---

## What This Is

A step-by-step guide and reference for building a custom Windows installation image (`install.wim`) from scratch. It covers:

- Downloading a stock Windows ISO from UUPDump
- Integrating .NET Framework 3.5 during the build
- Selecting editions and architectures
- Setting up a Hyper-V virtual machine for image customization
- Booting into audit mode with a provided `autounattend.xml`
- Applying customizations (applications, drivers, registry, defaults)
- Sealing the image with `sysprep /generalize`
- Capturing the sealed image with `dism /Capture-Image`
- Importing the captured image into MDT or any other deployment method

**This repository is a companion to [`MDT-Zero-Touch-Deployment`](https://github.com/ArthurJDurand/MDT-Zero-Touch-Deployment).** That project expects a Windows `install.wim` to import into MDT. You can obtain one from Microsoft, your organization, or by building your own with the workflow described here. Building your own gives you:

- **Always current builds** — no waiting for someone else to refresh a pre-built image
- **Custom editions** — pick exactly which editions ship in the ISO
- **.NET 3.5 integration** — baked in during the build, no post-deployment step
- **Full control** — audit mode customizations are applied before deployment, so every target gets the same baseline
- **No dependencies** — no personal cloud storage links that may disappear

### Who This Is For

- Technicians deploying Windows with `MDT-Zero-Touch-Deployment` who want their own custom image
- Administrators building reference images for any deployment method (MDT, SCCM, Intune, Autopilot, WDS)
- Anyone who needs a Windows image with specific prerequisites, applications, or configuration already applied

### What This Is Not

- Not a deployment tool. This repository produces an image; it does not deploy it.
- Not a Windows modification suite. It does not remove components, strip features, or de-bloat the OS.
- Not a legal shortcut. You still need a valid Windows license for any image you build.

---

## Workflow

```
1. Download UUPDump package for the chosen Windows version
2. Run the UUPDump script to build the ISO
3. Extract install.wim (or install.esd) from the ISO
4. Set up a Hyper-V VM with the ISO or the extracted WIM
5. Boot into audit mode with the provided autounattend.xml
6. Apply customizations in audit mode
7. Seal the image with sysprep /generalize /oobe /shutdown
8. Capture the sealed image with dism /Capture-Image
9. Import into MDT or another deployment tool
```

Steps 1–3 are covered in this README's [Quick Start](#quick-start). Steps 4–9 are covered in detail under [docs/](docs/) and expanded on in the [Audit Mode Workflow](#audit-mode-workflow) section.

---

## Quick Start

### 1. Clone this repository

```bash
git clone https://github.com/ArthurJDurand/MDT-Windows-Image-Builder.git
```

### 2. Read the full guide

The complete workflow is in [docs/WINDOWS-IMAGE-BUILDING.md](docs/WINDOWS-IMAGE-BUILDING.md).

### 3. Install prerequisites

- **7-Zip** — for extracting the UUPDump package and the ISO
- **Windows ADK for Windows 11** — provides DISM, `oscdimg`, and other deployment tools
- **A Windows build host** — Windows 10 or Windows 11 with at least 30 GB free disk space
- **(Optional) Hyper-V** — for the audit-mode customization step

### 4. Build the ISO

Open [uupdump.net](https://uupdump.net/):

1. Search for the Windows version you want (Windows 11 or Windows 10)
2. Choose the latest stable build
3. Select **Pro** edition (uncheck editions you do not need to reduce ISO size)
4. Choose your language and architecture
5. On the conversion options screen, check **Integrate .NET Framework 3.5**
6. Click **Create download package**
7. Extract the downloaded zip
8. Run `uup_download_windows.cmd`

The script downloads components from Microsoft and builds the ISO. Expect 30–60 minutes.

### 5. Extract or import

Either import the ISO directly into MDT, or extract `install.wim` from the ISO's `sources\` folder. If the ISO ships `install.esd`, convert it:

```powershell
dism /Export-Image /SourceImageFile:C:\path\to\install.esd /SourceIndex:1 /DestinationImageFile:C:\path\to\install.wim /Compress:max /CheckIntegrity
```

### 6. Customize (optional)

If you want a customized image (specific applications, drivers, registry settings, or a preconfigured default user profile), follow the audit-mode workflow in [docs/AUDIT-MODE.md](docs/AUDIT-MODE.md).

### 7. Import into MDT

Import the ISO or the extracted/captured WIM into your MDT deployment share. See [`MDT-Zero-Touch-Deployment` docs/SETUP.md](https://github.com/ArthurJDurand/MDT-Zero-Touch-Deployment/blob/main/docs/SETUP.md) for the full workflow.

---

## Audit Mode Workflow

Audit mode is a Windows Setup mode that boots directly to a full desktop running as the built-in Administrator, with `sysprep` available. It is designed for OEMs and image builders — you apply customizations, then seal the image for reuse.

The workflow in this repository:

1. Boot the reference machine (or a Hyper-V VM) from a Windows ISO
2. Use the provided `autounattend.xml` to skip OOBE and enter audit mode automatically
3. Apply your customizations — install applications, add drivers, set registry keys, configure the default user profile
4. Run `sysprep /generalize /oobe /shutdown` to seal the image
5. Capture the sealed image with `dism /Capture-Image`

The full guide, including the `autounattend.xml` files and step-by-step instructions, is in [docs/AUDIT-MODE.md](docs/AUDIT-MODE.md).

---

## Repository Structure

```
MDT-Windows-Image-Builder/
├── README.md
├── LICENSE.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── .gitignore
├── .github/
│   ├── FUNDING.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   └── feature_request.yml
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── release.yml
├── autounattend/
│   ├── audit-mode.xml            Bypasses OOBE and enters audit mode
│   └── README.md                 How to use these files
├── scripts/
│   ├── New-WindowsISO.ps1        Wrapper around UUPDump workflow
│   ├── Mount-InstallWim.ps1      Mount install.wim for offline changes
│   ├── Capture-WindowsImage.ps1  Capture a sysprepped image
│   └── README.md
└── docs/
    ├── WINDOWS-IMAGE-BUILDING.md The full UUPDump workflow
    ├── AUDIT-MODE.md             Audit mode customization guide
    ├── HYPER-V-SETUP.md          VM setup for image building
    ├── SYSPREP.md                Sysprep pitfalls and best practices
    ├── CAPTURE-AND-IMPORT.md     Capturing and importing the image
    ├── TROUBLESHOOTING.md        Common errors and fixes
    └── images/                   Screenshots and diagrams
```

Not every file above exists yet. See [Documentation](#documentation) for what is currently available.

---

## Documentation

| Document | Description | Status |
|---|---|---|
| [docs/WINDOWS-IMAGE-BUILDING.md](docs/WINDOWS-IMAGE-BUILDING.md) | Full UUPDump workflow — download, integrate .NET 3.5, choose editions, build ISO | Planned |
| [docs/AUDIT-MODE.md](docs/AUDIT-MODE.md) | Audit mode customization guide — enter audit mode, apply customizations, seal with sysprep | Planned |
| [docs/HYPER-V-SETUP.md](docs/HYPER-V-SETUP.md) | Hyper-V virtual machine setup for image building | Planned |
| [docs/SYSPREP.md](docs/SYSPREP.md) | Sysprep pitfalls and best practices | Planned |
| [docs/CAPTURE-AND-IMPORT.md](docs/CAPTURE-AND-IMPORT.md) | Capturing the sealed image and importing into MDT | Planned |
| [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | Common errors and fixes | Planned |
| [CHANGELOG.md](CHANGELOG.md) | Version history and notable changes | Planned |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute | Planned |

This repository is in early development. The README is the current reference; detailed docs will follow as the workflow is validated end-to-end.

---

## Requirements

### Build host

- Windows 10 or Windows 11
- At least 30 GB free disk space
- 7-Zip installed at `C:\Program Files\7-Zip\7z.exe`
- Windows ADK for Windows 11 (for DISM and image tools)
- Internet access to reach uupdump.net and Microsoft's CDN

### Optional — for audit-mode customization

- Hyper-V enabled (requires Windows 10/11 Pro, Enterprise, or Education)
- At least 8 GB RAM for the VM
- At least 60 GB of virtual disk space for the VM

### Optional — for capturing the sealed image

- Windows PE boot media (from the ADK)
- A second disk or network share to store the captured WIM

---

## Related Projects

| Project | Purpose |
|---|---|
| [`MDT-Zero-Touch-Deployment`](https://github.com/ArthurJDurand/MDT-Zero-Touch-Deployment) | The MDT deployment share that consumes the images built with this repository. Includes task sequences, OEM driver injection, an OEM Apps framework, and offline media support. |

---

## Contributing

Contributions are welcome. Areas where help is especially valuable:

- **Validating the workflow on different Windows builds** — if you build successfully on a build not yet documented, report it
- **Sysprep tips and pitfalls** — real-world issues from your deployments
- **Screenshots** — Hyper-V setup, audit mode, and sysprep dialogs
- **Alternative approaches** — if you have a workflow that avoids UUPDump or audit mode and works better, open a discussion
- **Documentation** — corrections, clarifications, and expanded examples

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full contribution guidelines (planned).

---

## Support This Project

This project is maintained in spare time and provided free of charge. If it saved you or your organization time, consider [sponsoring ongoing maintenance](https://github.com/sponsors/ArthurJDurand). Sponsorship funds documentation, testing across hardware, and issue triage.

[![Sponsor](https://img.shields.io/badge/Sponsor-ArthurJDurand-ea4aaa?logo=github-sponsors&logoColor=white)](https://github.com/sponsors/ArthurJDurand)

---

## License

MIT License — see [LICENSE.md](LICENSE.md).

---

## Author

**Arthur Durand**

- GitHub: [@ArthurJDurand](https://github.com/ArthurJDurand)
- Repository: [MDT-Windows-Image-Builder](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder)
- Sponsor: [github.com/sponsors/ArthurJDurand](https://github.com/sponsors/ArthurJDurand)

---

## Acknowledgments

- **UUPDump** ([uupdump.net](https://uupdump.net/)) — the tool that makes building stock Windows ISOs possible without a Microsoft account or VLSC access
- **Microsoft** — for the Windows ADK, DISM, and the audit mode workflow
- The MDT and Windows deployment communities for ongoing knowledge sharing

---

<div align="center">

**If this project helped you build a Windows image, consider giving it a ⭐**

</div>
