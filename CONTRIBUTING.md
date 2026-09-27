# Contributing to MDT Windows Image Builder

First off — **thank you** for considering a contribution. This project exists because the Windows deployment community shares knowledge, and every improvement (a clarified step, a fixed script, a new autounattend template) makes it more useful for everyone.

This document explains how to contribute effectively and what standards your contributions should meet.

---

## Table of Contents

- [Ways to Contribute](#ways-to-contribute)
- [Before You Start](#before-you-start)
- [Development Setup](#development-setup)
- [Script Standards](#script-standards)
- [Autounattend Standards](#autounattend-standards)
- [Documentation Standards](#documentation-standards)
- [Commit Message Convention](#commit-message-convention)
- [Branch Naming](#branch-naming)
- [Pull Request Process](#pull-request-process)
- [What Reviewers Look For](#what-reviewers-look-for)
- [Working Across the Three Repositories](#working-across-the-three-repositories)
- [Areas Where Help Is Needed](#areas-where-help-is-needed)
- [Reporting Bugs](#reporting-bugs)
- [License of Contributions](#license-of-contributions)
- [Code of Conduct](#code-of-conduct)
- [Questions](#questions)

---

## Ways to Contribute

You don't have to write code to contribute. All of the following are valuable:

| Contribution Type | Examples |
|---|---|
| **Workflow validation** | Confirm the workflow works on a Windows build we have not tested, report what changed |
| **Documentation** | Clarify a step, fix a typo, add screenshots of key screens |
| **Autounattend templates** | New variants for specific scenarios (Hyper-V, physical, no TPM, and so on) |
| **Scripts** | A new wrapper, a fix to an existing script, a portable version |
| **Sysprep knowledge** | Real-world pitfalls, hardware-specific issues, tips from your deployments |
| **Alternative approaches** | A workflow that avoids UUPDump or audit mode and works better in some cases |
| **Ideas** | Feature requests, workflow suggestions, architectural feedback |

If you're unsure whether an idea is in scope, **open a [Discussion](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/discussions) first** before investing time in a PR.

---

## Before You Start

### Check existing issues and PRs

Someone may already be working on the same thing. Search [open issues](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/issues) and [open PRs](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/pulls) before starting.

### Open an issue first for large changes

For anything beyond a typo or obvious fix — especially new scripts, new autounattend files, or workflow changes — **open an issue to discuss the approach first**. This avoids wasted effort if the change does not fit the project's direction.

### Small, focused PRs are preferred

One logical change per PR. A PR that fixes a typo *and* adds a script *and* reformats XML is hard to review and hard to revert if something goes wrong.

---

## Development Setup

To test your changes properly, you need a build environment.

### Minimum requirements

- **Windows 10 or Windows 11 build host** with at least 30 GB free disk space
- **7-Zip** installed at `C:\Program Files\7-Zip\7z.exe`
- **Windows ADK for Windows 11** — provides DISM, `oscdimg`, and other deployment tools
- **Internet access** to reach uupdump.net and Microsoft's CDN
- **(For audit mode testing) Hyper-V enabled** or a spare physical machine

### Recommended workflow

1. Fork the repository
2. Clone your fork:
   ```bash
   git clone https://github.com/<your-username>/MDT-Windows-Image-Builder.git
   ```
3. Make your changes
4. Test end-to-end — this is the important part:
   - Build an ISO from UUPDump
   - Boot a VM or physical machine
   - Complete the workflow through to image capture
   - Import the captured image into MDT or another deployment tool
5. Commit and push to your fork
6. Open a PR against the `main` branch

### Testing requirements by change type

| Change Type | Testing Required |
|---|---|
| Documentation only | None, but proofread carefully |
| Typo or clarification fix | None, but ensure the fix does not change meaning |
| Script bug fix | Reproduce the bug, apply the fix, verify it is resolved |
| New script | Test end-to-end on at least one Windows build |
| Autounattend change | Boot a fresh VM with the change and confirm it reaches the intended state |
| Workflow change | Full end-to-end build and capture, plus a deployment test with the captured image |
| Windows version support | Full workflow on the new Windows version |

**State what you tested on in the PR description.** "Tested on Windows 11 24H2 build 26100.1742, Hyper-V Gen 2, amd64 Pro" is far more useful than "tested and works."

---

## Script Standards

PowerShell scripts live in `scripts/`. They automate parts of the workflow.

### 1. Header documentation

Every script must begin with a comment block containing at minimum:

```powershell
<#
.SYNOPSIS
    One-line description of what the script does.

.DESCRIPTION
    Longer explanation including when this script runs in the workflow
    and what preconditions it expects.

.PARAMETER <Name>
    Description of each parameter.

.NOTES
    - External dependencies (7-Zip, ADK, Hyper-V)
    - Admin rights required
    - Known limitations
#>
```

### 2. Parameter blocks

Use `[CmdletBinding()]` and explicit parameter definitions. Avoid positional-only arguments for anything non-obvious.

```powershell
[CmdletBinding()]
param(
    [Parameter(Mandatory = $true)]
    [string]$IsoPath,

    [Parameter(Mandatory = $false)]
    [string]$OutputPath = "C:\Images"
)
```

### 3. Path handling

Never hardcode paths that belong to the user. Accept them as parameters with sensible defaults.

```powershell
# BAD
$output = "C:\Users\Arthur\Images\build.wim"

# GOOD
$output = if ($OutputPath) { $OutputPath } else { Join-Path $env:TEMP "build.wim" }
```

### 4. Retry logic for DISM and long-running operations

DISM and robocopy fail intermittently. Wrap them in retry loops.

```powershell
$maxAttempts = 3
for ($attempt = 1; $attempt -le $maxAttempts; $attempt++) {
    & dism.exe /Mount-Image /ImageFile:$WimPath /Index:1 /MountDir:$MountDir
    if ($LASTEXITCODE -eq 0) { break }
    if ($attempt -lt $maxAttempts) { Start-Sleep -Seconds 5 }
}
```

### 5. Preserve exit codes

Never mask a failure with a silent `try/catch` that swallows the error. If a script fails, the caller should know.

```powershell
# BAD — swallows the error
try { & dism.exe /Capture-Image ... } catch { }

# GOOD — check the exit code explicitly
& dism.exe /Capture-Image ...
if ($LASTEXITCODE -ne 0) {
    throw "DISM capture failed with exit code $LASTEXITCODE"
}
```

### 6. Cleanup on failure

Any script that mounts a WIM, attaches a VHD, or creates a resource that must be released should use `try/finally` to guarantee cleanup.

```powershell
try {
    & dism.exe /Mount-Image /ImageFile:$WimPath /Index:1 /MountDir:$MountDir
    # ... do work ...
}
finally {
    & dism.exe /Unmount-Image /MountDir:$MountDir /Discard
}
```

### 7. Progress output

Long-running operations should report progress. Use `Write-Progress` for steps that take more than a few seconds, or `Write-Host` for stage announcements.

```powershell
Write-Progress -Activity "Building ISO" -Status "Extracting install.wim" -PercentComplete 40
```

### 8. PowerShell 5.1 compatibility

Scripts must run under the Windows version of PowerShell, which is **5.1**. Do not use syntax or cmdlets exclusive to PowerShell 7 (ternary operator `? :`, `??`, `-Parallel`) unless the script explicitly requires PS7 and documents that in its header.

### 9. No `exit` in library scripts

Scripts meant to be dot-sourced or called from another script should **return** rather than `exit`, so the caller can inspect the result.

Standalone scripts may `exit` with a code.

### 10. Idempotence

Scripts that can be re-run should be idempotent. For example, a script that extracts a WIM should skip the extraction if the target folder already contains the expected files.

---

## Autounattend Standards

`autounattend.xml` files in `autounattend/` control how Windows Setup behaves. These are the most sensitive files in the repository because a mistake can leave a user stuck at an OOBE screen with no obvious error.

### 1. Validate before committing

Every autounattend file must be well-formed XML:

```powershell
Get-Content .\autounattend\audit-mode.xml -Raw | [xml] | Out-Null
if ($?) { "XML is valid" } else { "XML is invalid" }
```

### 2. Document each component

Add comments inside the XML explaining what each component does and why. Reviewers and future maintainers should not have to look up Microsoft documentation to understand the file.

```xml
<!-- Bypass OOBE and enter audit mode automatically -->
<component name="Microsoft-Windows-Deployment" ...>
  ...
</component>
```

### 3. No personal values

Never commit an autounattend that contains:

- Product keys
- Credentials or passwords
- Machine-specific names
- Personal organization names

If a template needs a placeholder, use an obvious one like `YOUR-PRODUCT-KEY` or `COMPANY-NAME` and document it in the header.

### 4. Locale placeholders

Locale and timezone values should be clearly replaceable. Provide a comment block at the top listing every value the user must review:

```xml
<!--
  Locale defaults. Change these to match your environment:
    InputLocale  — en-US
    SystemLocale — en-US
    UILanguage   — en-US
    UserLocale   — en-ZA
    TimeZone     — South Africa Standard Time
-->
```

### 5. Test on the target Windows build

An autounattend that works on Windows 11 23H2 may not work on 24H2. Always test on the specific build the template claims to support, and state that build in the file header.

### 6. Do not remove existing entries

If you are adding to an existing autounattend file, do not silently remove entries that other users may rely on. Add, comment out with explanation, or open a Discussion first.

---

## Documentation Standards

### 1. Structure

Every documentation page should start with a short paragraph describing what the page covers and who it is for. Then a Table of Contents if the page is longer than a few hundred lines.

### 2. Commands

Wrap commands in fenced code blocks with the correct language tag (`powershell`, `cmd`, `bash`, `xml`, `yaml`, `markdown`).

### 3. Screenshots

Screenshots live in `docs/images/`. Name them descriptively: `hyper-v-gen2-settings.png`, not `screenshot1.png`. Keep file sizes reasonable (compress PNGs).

### 4. Cross-references

Link between docs using relative paths: `[docs/AUDIT-MODE.md](AUDIT-MODE.md)`, not absolute URLs. This keeps the docs working if the repo moves.

### 5. Version-specific notes

If a step differs between Windows versions, use a table or clearly labeled subsections. Do not mix guidance for 22H2 and 24H2 in the same paragraph.

### 6. Update the README

If your documentation change affects the workflow overview, update the README's workflow diagram or table.

---

## Commit Message Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/). This makes the changelog easier to generate and clarifies what each commit does.

### Format

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

### Types

| Type | Use For |
|---|---|
| `feat` | A new feature, script, template, or doc |
| `fix` | A bug fix |
| `docs` | Documentation changes only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `chore` | Maintenance (dependency bumps, formatting, build changes) |
| `revert` | Reverting a previous commit |

### Scopes (optional but recommended)

- `uupdump`, `audit-mode`, `sysprep`, `capture`, `hyper-v`, `dism`, `wim`
- `scripts`, `autounattend`, `templates`
- `docs`, `readme`, `changelog`

### Examples

```
feat(uupdump): add PowerShell wrapper for ISO download
fix(audit-mode): correct LocalAccount entry for Windows 11 24H2
docs(sysprep): clarify generalization step for Hyper-V
feat(audit-mode): add autounattend variant without TPM requirement
```

### Breaking changes

If a change requires users to rebuild images or update their autounattend files, add `!` after the type/scope and include a `BREAKING CHANGE:` footer:

```
feat(autounattend)!: switch audit mode entry to a new registry path

BREAKING CHANGE: The audit-mode.xml file now uses a new registry
entry that requires Windows 11 24H2 or newer. Users on 23H2 must
use the previous template version.
```

---

## Branch Naming

Use a prefix that matches the commit type:

| Prefix | Use For |
|---|---|
| `feature/` | New features, scripts, templates, or docs |
| `fix/` | Bug fixes |
| `docs/` | Documentation changes |
| `refactor/` | Code refactoring |
| `chore/` | Maintenance |

Examples:

- `feature/powershell-uupdump-wrapper`
- `fix/audit-mode-24h2-localaccount`
- `docs/sysprep-troubleshooting`
- `feature/hyper-v-gen2-template`

---

## Pull Request Process

1. **Fork** the repository
2. **Create a branch** from `main` using the naming convention above
3. **Make your changes** following the script, autounattend, and documentation standards
4. **Update `CHANGELOG.md`** — add your change under `[Unreleased]` in the appropriate section (`Added`, `Changed`, `Fixed`, `Removed`, `Security`)
5. **Test end-to-end** where applicable — especially for scripts and autounattend changes
6. **Push** to your fork
7. **Open a PR** against `main` with:
   - A clear title matching the Conventional Commits format
   - A description of **what** changed and **why**
   - The **Windows build** and **environment** you tested on
   - A **link to the related issue** if one exists
   - Screenshots or log excerpts if relevant

### PR title format

Match your commit message convention:

```
feat(uupdump): add PowerShell wrapper for ISO download
```

### What makes a good PR description

```markdown
## Summary
Adds a PowerShell wrapper that automates the UUPDump download and
build process, so the workflow can be scripted.

## Changes
- Added scripts/New-WindowsISO.ps1
- Documented the script in docs/WINDOWS-IMAGE-BUILDING.md
- Updated README to reference the new script

## Testing
- Built ISO from UUPDump build 26100.1742 (Windows 11 24H2, Pro, amd64)
- Script completed without manual intervention in 42 minutes
- Resulting ISO verified against the SHA-256 on the UUPDump page
- Booted the ISO in a Hyper-V Gen 2 VM and completed the audit-mode workflow

## Related Issue
Closes #12

## Checklist
- [x] Follows script standards
- [x] CHANGELOG.md updated under [Unreleased]
- [x] Tested end-to-end
- [x] No personal product keys or credentials in any file
- [x] PowerShell 5.1 compatible
```

---

## What Reviewers Look For

When reviewing a PR, the maintainer checks:

| Item | Why |
|---|---|
| **End-to-end testing** | A workflow change is only valid if it has been tested through image capture |
| **Windows build stated** | Behaviour differs between 22H2, 23H2, and 24H2 |
| **Autounattend XML validity** | Malformed XML leaves the user stuck at OOBE |
| **No secrets in autounattend** | Product keys and credentials must never be committed |
| **Script header documentation** | SYNOPSIS, DESCRIPTION, PARAMETER, NOTES block present |
| **Parameter blocks** | No hardcoded user paths |
| **Cleanup on failure** | WIMs unmounted, VHDs detached, in `finally` blocks |
| **Exit code preservation** | Failures are visible to the caller |
| **PowerShell 5.1 compatible** | Scripts must run on stock Windows |
| **Documentation updated** | README and relevant docs reflect the change |
| **CHANGELOG entry** | Added under `[Unreleased]` |
| **Small, focused scope** | One logical change per PR |

---

## Working Across the Three Repositories

This project is part of a three-repository ecosystem. Before opening a PR, determine which repository the change belongs to.

| Repository | What belongs there |
|---|---|
| [`MDT-Zero-Touch-Deployment`](https://github.com/ArthurJDurand/MDT-Zero-Touch-Deployment) | Task sequences, deployment scripts, the OEM Apps framework, the deployment share structure, offline media workflow, and all deployment documentation |
| [`MDT-OEM-Extensibility`](https://github.com/ArthurJDurand/MDT-OEM-Extensibility) | The tooling and recipes that build the per-vendor `.7z` payload archives. Changes to how OEM apps are downloaded, staged, and packed belong there. |
| [`MDT-Windows-Image-Builder`](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder) (this repo) | The UUPDump workflow, `autounattend.xml` templates, Hyper-V setup, audit mode, and image capture. Changes to how Windows images are built belong here. |

If you are unsure, open a Discussion. Cross-repository changes should be coordinated across the relevant repositories.

### What this repository does not cover

- **Deployment** — how an image is installed, driver-injected, or customized at OOBE belongs in `MDT-Zero-Touch-Deployment`
- **OEM payload construction** — how vendor apps and drivers are downloaded and packed belongs in `MDT-OEM-Extensibility`
- **Task sequences** — the sequence definitions that deploy an image belong in `MDT-Zero-Touch-Deployment`

Keep the scope of this repository focused on producing the `install.wim` and any audit-mode customizations. Everything downstream is another repository's responsibility.

---

## Areas Where Help Is Needed

Some specific things the maintainer would love help with:

### 1. Workflow validation on different Windows builds

The workflow has been tested on a limited set of Windows builds. If you successfully build an image on a build not yet documented, report it in a Discussion or PR with:

- The UUPDump build ID
- The Windows version and edition
- The environment (Hyper-V, physical, other)
- Any deviations from the documented steps

### 2. Sysprep knowledge from real deployments

Sysprep is the most common source of image-building failures. Real-world pitfalls are more valuable than anything else in this repository. If you have encountered and solved sysprep issues, please share:

- The error message
- The Windows build
- The cause
- The fix

### 3. Autounattend variants

The current `audit-mode.xml` targets a general case. Useful variants:

- No TPM requirement (physical machines without TPM 2.0)
- Single-language install (Windows 11 Home Single Language)
- Enterprise edition
- Physical machine without Hyper-V (bare metal audit mode)

### 4. Alternative approaches

If you have a workflow that avoids UUPDump or audit mode and works better for some scenarios, open a Discussion. Alternatives worth documenting:

- Fully unattended builds using only DISM and no VM
- Batch automation using NTLite or similar tools
- Zero-touch image builds with CI runners

### 5. Script improvements

- Cross-platform PowerShell (7.x) variants for Linux and macOS build hosts
- Better error reporting and structured logging
- Dry-run modes for destructive scripts

### 6. Documentation

- Screenshots of Hyper-V, audit mode, and sysprep dialogs
- A "known working builds" table showing which Windows builds have been validated
- A troubleshooting decision tree for common sysprep errors

If any of these interest you, **open a Discussion first** so we can scope it together.

---

## Reporting Bugs

Found a bug? [Open an issue](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/issues/new) with:

- **Description** — what you expected vs. what happened
- **Steps to reproduce** — exact steps, commands, and files involved
- **Windows build** — the specific build number of the ISO you built
- **Environment** — Hyper-V, physical, VMware, VirtualBox, or other
- **Relevant log file** — Panther logs, DISM logs, or script output
- **Screenshots** if applicable

**Please don't paste full logs inline** — attach them as files or link to a Gist.

---

## License of Contributions

By submitting a pull request to this project, you agree that your contribution is licensed under the same [MIT License](LICENSE) that governs the project.

You confirm that:

- You have the right to submit the contribution
- The contribution is your original work, or you have obtained permission to submit it under the MIT License
- Any third-party code, scripts, or XML included in your contribution is compatible with the MIT License and clearly attributed
- Your contribution does not include product keys, credentials, or other secrets that are not your own to share

---

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/).

In short: be respectful, be patient, assume good faith, and focus on the technical problem. Harassment, personal attacks, and dismissive behavior are not tolerated. Violations can be reported to the maintainer via a [private GitHub security advisory](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/security/advisories/new).

---

## Questions?

- **General questions or ideas:** [GitHub Discussions](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/discussions)
- **Bug reports:** [GitHub Issues](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/issues)
- **Security vulnerabilities:** [Private security advisory](https://github.com/ArthurJDurand/MDT-Windows-Image-Builder/security/advisories/new)
- **Direct contact:** See the [author's GitHub profile](https://github.com/ArthurJDurand)

---

## Thank You

Whether you validate the workflow on a build we have not tested, share a sysprep fix, or clarify a step in the docs — **your contribution matters**. Every improvement helps someone build a Windows image with less friction.

Thank you for being part of it.

---

<div align="center">

**Happy building!** 🛠️

</div>
