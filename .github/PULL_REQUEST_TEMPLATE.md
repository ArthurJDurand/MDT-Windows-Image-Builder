<!--
  Thanks for submitting a pull request! Before you continue, please:

  1. Read CONTRIBUTING.md — especially the Script Standards section.
  2. Ensure your branch is based on the latest `main`.
  3. Make sure all applicable checkboxes below are ticked.

  For large or architectural changes, please open an issue or a Discussion
  first so we can agree on the approach before you invest significant time.
-->

## Summary

<!--
  One or two sentences describing WHAT this PR changes and WHY.
  Link the related issue if one exists.
-->

Closes #

## Type of Change

<!--
  Tick all that apply. This informs the reviewer and the auto-generated
  release notes (see .github/release.yml).
-->

- [ ] 🐛 Bug fix (non-breaking change that fixes an issue)
- [ ] 🚀 New feature (non-breaking change that adds functionality)
- [ ] ⚠️ Breaking change (fix or feature that changes the workflow; may require users to rebuild images or update their autounattend files)
- [ ] 📖 Documentation only (no script or template changes)
- [ ] 🧩 Autounattend template change
- [ ] 🧹 Refactor or maintenance (no functional change)
- [ ] 🔒 Security fix

## Affected Area

<!--
  Which parts of the project does this PR touch? Tick all that apply.
  This helps reviewers focus and helps users know what to re-test.
-->

- [ ] `autounattend/` — autounattend XML templates
- [ ] `scripts/` — PowerShell scripts (ISO build, DISM, capture)
- [ ] `templates/` — XML and JSON configuration templates
- [ ] `docs/` — documentation
- [ ] `README.md` — project README
- [ ] Repository infrastructure (`.github/`, `LICENSE.md`, `CHANGELOG.md`, `.gitignore`)

## Changes Made

<!--
  A bullet list of the specific changes. Keep it scannable.
  Reference specific files or functions where relevant.
-->

-
-
-

## Testing

<!--
  Describe exactly how you tested this. "It works" is not enough.
  The more detail, the easier it is to review with confidence.
-->

**Windows version and build tested:**

<!--
  Example: Windows 11 24H2 (build 26100.1742), Pro edition, amd64
-->

**Build environment:**

<!-- Tick all that apply -->

- [ ] Physical machine — Hyper-V enabled
- [ ] Physical machine — Hyper-V not used
- [ ] Nested virtualization (Hyper-V inside another hypervisor)
- [ ] VMware Workstation / Player
- [ ] VirtualBox
- [ ] QEMU / KVM
- [ ] Other (specify below)

**Build host operating system:**

<!--
  Example: Windows 11 Pro 23H2 (build 22631.4460)
-->

**Testing steps performed:**

<!--
  Example:
  1. Built ISO from UUPDump build 26100.1742 with .NET 3.5 integration enabled.
  2. Created Hyper-V Gen 2 VM with 4 GB RAM and 60 GB disk.
  3. Booted with the modified autounattend/audit-mode.xml at ISO root.
  4. Verified the VM entered audit mode automatically.
  5. Applied customizations and ran sysprep /generalize /oobe /shutdown.
  6. Captured the image with dism /Capture-Image.
  7. Imported the captured WIM into MDT and deployed to a test machine.
-->

1.
2.
3.

**Result:**

<!--
  Example: Workflow completed successfully end-to-end. The captured image
  deployed cleanly through MDT and all framework phases converged.
-->

## Known Issues / Limitations

<!--
  Anything that doesn't work, needs follow-up, or is known to be a
  partial implementation. Be honest — reviewers would rather know now.
-->

- None.

## Screenshots or Logs

<!--
  If this PR changes a visible workflow step (audit mode screen, sysprep
  dialog, DISM output), include before/after evidence. Drag and drop images
  directly into this box. For logs, attach as a file or upload to a Gist —
  do not paste long logs inline.
-->

## Checklist

<!--
  Every box must be ticked before a reviewer will look at the PR.
  If a box doesn't apply, explain why in the Notes section below.
-->

### Script Standards (see CONTRIBUTING.md)

- [ ] PowerShell scripts include a `.SYNOPSIS` / `.DESCRIPTION` / `.NOTES` header block
- [ ] Scripts use `Select-Object -First 1` or equivalent defensive handling where results could be arrays
- [ ] Long-running operations (DISM, robocopy, ISOs) include retry logic or clear failure reporting
- [ ] Scripts preserve `$LASTEXITCODE` and surface failures to the caller — no silent `try/catch` swallowing
- [ ] No hardcoded paths to the user's disk, user profile, or VM — paths are parameterized or discovered
- [ ] Scripts are PowerShell 5.1 compatible (no PS7-only syntax) unless explicitly documented otherwise
- [ ] No `exit` in scripts intended to be dot-sourced or called from another script
- [ ] Scripts that modify a mounted WIM include proper `Dismount-WindowsImage` cleanup in a `finally` block

### Autounattend Templates

- [ ] The XML is valid and passes `Get-Content <file> -Raw | [xml]` without errors
- [ ] Locale, timezone, and input locale placeholders are documented and easy to change
- [ ] No personal product keys, credentials, or machine-specific values are present
- [ ] The template has been tested on the Windows build it targets

### Repository Hygiene

- [ ] `CHANGELOG.md` updated under `[Unreleased]` in the appropriate section (`Added`, `Changed`, `Fixed`, `Removed`, `Security`)
- [ ] No large binaries committed (ISOs, WIMs, VM disks, captured images)
- [ ] No credentials, product keys, or secrets committed
- [ ] No unrelated files, formatting changes, or drive-by refactors included
- [ ] Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat(scope): …`, `fix(scope): …`, etc.)
- [ ] Branch is based on the latest `main` and rebased if necessary
- [ ] PR title matches the Conventional Commits format

### Documentation

- [ ] User-facing documentation updated (`README.md` or `docs/*`) if the workflow changed
- [ ] If a new script was added, it is described in the README's repository structure and referenced from the relevant doc
- [ ] If a new autounattend file was added, `autounattend/README.md` describes its purpose
- [ ] If a new workflow step was added, the README's workflow diagram is updated
- [ ] Screenshots or terminal output included for workflow changes where helpful

### Testing

- [ ] Tested end-to-end on a fresh Windows ISO build (not just a partial run)
- [ ] Tested in the environment the change is intended for (Hyper-V, physical hardware, or another hypervisor)
- [ ] For autounattend changes, verified the VM reaches the intended state without user interaction
- [ ] No regressions observed in adjacent workflow steps

## Notes for Reviewers

<!--
  Anything else the reviewer should know. Call out tricky parts, areas
  you're unsure about, or questions you'd like feedback on.
-->

- None.

---

<!--
  By submitting this pull request, you agree that your contribution is
  licensed under the MIT License that governs this project. See
  CONTRIBUTING.md → "License of Contributions" for details.
-->
