# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## Versioning Policy

Because this repository documents a workflow rather than shipping code that runs in production, version numbers have specific meaning for users:

| Bump | Meaning | Examples |
|---|---|---|
| **MAJOR** | Breaking changes to the workflow that require users to rebuild images or update their `autounattend.xml` files. Existing images built from an older workflow are still valid but new builds must follow the new steps. | A new autounattend structure, a change in the audit-mode entry method, a step removed or replaced |
| **MINOR** | New workflow steps, new templates, new scripts, or new documentation that are backward-compatible. Existing images remain valid and users can adopt the new content at their convenience. | A new script, a new autounattend variant, support for a new Windows build |
| **PATCH** | Corrections to documentation, typo fixes, and clarifications. Safe to pull at any time. | Fixed commands in a doc, corrected screenshots, clarified sysprep notes |

**When upgrading across MAJOR versions, read the migration notes in the release description.**

---

## [Unreleased]

### Added

### Changed

### Fixed

### Removed

### Security

---

## [1.0.0] - YYYY-MM-DD

Initial public release.

### Added

#### Documentation

- `README.md` — Project overview, workflow summary, quick start, and repository structure.
- `LICENSE.md` — MIT License.
- `CODE_OF_CONDUCT.md` — Contributor Covenant Code of Conduct.

#### Repository Infrastructure

- `.gitignore` — Ignore rules tailored for image building: excludes ISOs, WIMs, ESDs, SWMs, VM disk files, UUPDump working directories, DISM mount points, and logs.
- `.github/FUNDING.yml` — GitHub Sponsors configuration.
- `.github/release.yml` — Auto-categorized release notes from pull requests.
- `.github/ISSUE_TEMPLATE/bug_report.yml` — Bug report template tuned for image-building issues.
- `.github/ISSUE_TEMPLATE/feature_request.yml` — Feature request template.
- `.github/PULL_REQUEST_TEMPLATE.md` — Pull request template with checklists for scripts, autounattend files, and workflow validation.

#### Planned Content

The following are documented as part of the roadmap but are not yet shipped. They will be added as the workflow is validated end-to-end on real builds.

- `docs/WINDOWS-IMAGE-BUILDING.md` — Full UUPDump workflow: download, .NET 3.5 integration, edition selection, ISO build.
- `docs/AUDIT-MODE.md` — Audit mode customization guide.
- `docs/HYPER-V-SETUP.md` — Hyper-V virtual machine setup for image building.
- `docs/SYSPREP.md` — Sysprep pitfalls and best practices.
- `docs/CAPTURE-AND-IMPORT.md` — Capturing the sealed image and importing into MDT or another deployment tool.
- `docs/TROUBLESHOOTING.md` — Common errors and fixes.
- `autounattend/audit-mode.xml` — Unattend file that boots Windows Setup into audit mode.
- `scripts/New-WindowsISO.ps1` — Wrapper around the UUPDump workflow.
- `scripts/Mount-InstallWim.ps1` — Mount `install.wim` for offline servicing.
- `scripts/Capture-WindowsImage.ps1` — Capture a sysprepped image.

#### Related Projects

- Companion repository [`MDT-Zero-Touch-Deployment`](https://github.com/ArthurJDurand/MDT-Zero-Touch-Deployment) — the MDT deployment share that consumes the images built with this repository. Includes task sequences, OEM driver injection, an OEM Apps framework, and offline media support.

### Security

- Documented that `autounattend.xml` files must not contain personal product keys, credentials, or machine-specific values.
- The `.gitignore` excludes any file matching `*productkey*`, `*serial*`, and common secrets patterns to reduce accidental commits.

---

[Unreleased]: https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/releases/tag/v1.0.0
