# DeepSeek Harness — Windows Desktop

A personal Windows build of DeepSeek Harness, the plugin-based agent harness. You install one `.exe`, get a native window, and run agent sessions, agent teams, computer control, an in-app browser, and durable memory without a terminal. This repository publishes that desktop build and its install notes. It is not an official DeepSeek release.

## Table of contents

- [Download](#download)
- [Verify the file](#verify-the-file)
- [Install](#install)
- [First run](#first-run)
- [Where your data lives](#where-your-data-lives)
- [Uninstall](#uninstall)
- [What this build changes over the base](#what-this-build-changes-over-the-base)
- [Known limitations](#known-limitations)
- [Build it yourself](#build-it-yourself)
- [License and affiliation](#license-and-affiliation)

## Download

Take `DeepSeek-Harness-Setup.exe` from the latest [release](https://github.com/xiaobaga281/DSH-desktop/releases).

| Item | Value |
| --- | --- |
| File | `DeepSeek-Harness-Setup.exe` |
| Size | 554,730,507 bytes |
| App version | `0.1.6-alpha.1` |
| Windows file version | `0.1.6.1` |
| Target | x64, Windows 10 or later |

## Verify the file

Compare the digest before you run anything. Both commands were run against this build and return the value below.

```powershell
Get-FileHash -Algorithm SHA256 .\DeepSeek-Harness-Setup.exe
certutil -hashfile .\DeepSeek-Harness-Setup.exe SHA256
```

Expected SHA-256:

```text
06E7D00D357180820ABB5BBFD49AD1DB786095194FACE9AA13456CF6646834C5
```

A mismatch means the download is not this build. Do not install it.

## Install

Double-click the installer and follow the wizard, or run it without interaction. The installer is per-user, so it needs no administrator rights and installs under your profile by default.

```powershell
.\DeepSeek-Harness-Setup.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART
```

To pick another location, add `/DIR`. Both forms were run twice on this build's family: exit code 0, no elevation prompt, and no application start.

```powershell
.\DeepSeek-Harness-Setup.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /DIR="D:\DeepSeek-Harness"
```

What a silent install changes on your machine:

- It writes the application into the install directory and registers an entry under `HKCU\Software\Microsoft\Windows\CurrentVersion\Uninstall`.
- It creates a Start Menu entry inside `Programs\DeepSeek Harness\`.
- It does not create a desktop shortcut, because that task is off by default.
- It does not start the application.

## First run

A new install has no model provider, so configure one before your first message: open Settings, go to Models, and add a provider with its API address, protocol, and key.

The composer stays disabled and reads `选择一个工作区开始` (`Choose a workspace to start`) until you pick a workspace. That is the expected state of a fresh install, not a broken build.

## Where your data lives

The desktop build keeps everything under `%LOCALAPPDATA%\DeepSeek Harness`, and it never reads the data directory of a source checkout.

| Path | Contents |
| --- | --- |
| `dsh-home\settings.yaml` | Provider and application settings |
| `dsh-home\profiles` | Agent profiles and their installed packages |
| `dsh-home\skills`, `dsh-home\storages` | Skills and stored entities |
| `electron-user-data` | Chromium profile, cache, and the single-instance lock |
| `appearance`, `desktop-pet`, `notifications` | Background, pet, and notification preferences |

Your API key stays inside `dsh-home`. Copy that folder to move a configured install, and treat it as a secret.

## Uninstall

```powershell
& "$env:LOCALAPPDATA\Programs\DeepSeek Harness\unins000.exe" /VERYSILENT /SUPPRESSMSGBOXES /NORESTART
```

The uninstaller removes the application directory and its own Start Menu entry. It leaves a shortcut you created yourself, and it does not delete `dsh-home`, so your settings and keys survive an uninstall.

## What this build changes over the base

The base project ships a CLI and a plugin kernel. This build turns it into a desktop product. Measured on 2026-09-24 against a pristine copy of the same base version: 11,239 base files become 11,856, of which 440 are modified base files and 617 are added here. Nothing is deleted. The work is grouped into 40 capability surfaces.

| Area | What you get that the base does not do |
| --- | --- |
| Desktop shell | Native window with its own chrome, per-user installer, portable package layout, Windows toast identity that names the finished task, and a notification sound the renderer plays |
| Agent teams | Built-in team templates, an office view, a plan budget gate per stage, and per-stage owners |
| Session council | Peer sessions that exchange posts through `peer_*` tools and a council panel |
| Computer and browser control | A resident desktop driver for pointer and screen access, a control indicator, and an in-app browser panel with provider depth |
| Memory and decisions | Distillation, retrieval, and consolidation, plus 156 in-repo Agent Notes that record why each design choice was made |
| Chinese product surface | Localized settings shell with reordered sections, model catalog fetch, themes and backgrounds, a desktop pet, and archive semantics with permanent delete. The marketplace reads 12 built-in and 20 GitHub skills plus 165 MCP registry entries, and installs land in `dsh-home\skills` and a managed block of `cordis.patch.yml` |
| Media | Media tools and a long-video team that plans by form rather than a fixed clip length |
| Upgrades | A self-contained capsule so a new base version replays every customization through a real three-way merge, with three registration gates and one upgrade driver |

## Known limitations

- The 15-item desktop smoke run against this installed copy returns 14 pass, 0 fail, 1 skip once you have chosen a workspace (see [First run](#first-run)). The remaining skip is the tool-row check, which needs a session that has already called a tool.
- This build is x64 Windows only.
- It is a personal fork maintained alongside the base project, so expect no upgrade promise, no support channel, and no security advisory flow.

## Build it yourself

The desktop application is produced from a source checkout of this fork, not from this repository. From the project root:

```powershell
node scripts\ensure-workspace-links.mjs
powershell -File scripts\package-desktop.ps1
```

The packaging script runs the full build, resolves the dependency closure offline, stages the application, embeds the icon and version metadata, and compiles the installer with Inno Setup. It writes `dist-exe\DeepSeek-Harness-Setup.exe` and a portable ZIP next to it.

To move this fork onto a newer base version, run the upgrade driver instead of merging by hand:

```powershell
.\scripts\upgrade-deepseek-harness.cmd -DryRun -UpstreamRoot "<pristine official tree of the same version>"
```

It refuses to copy anything while the capsule and the customization ledger disagree.

## License and affiliation

This repository carries an Apache-2.0 `LICENSE` file for the material it publishes here: the install notes and the release assets. The underlying DeepSeek Harness project and its source remain governed by their own license and their own maintainers. This is not an official DeepSeek product, release, or endorsement.

## Dev Note

Numbers in this page come from measurements taken on 2026-09-24 on one x64 Windows machine: the installer digest, the silent install and uninstall runs, the file-count diff against a pristine base tree, and the desktop smoke run against the installed copy. The marketplace counts were read back from the installed copy, and one skill and one MCP server were installed through its own UI. The double-click wizard path and the desktop-shortcut task were not exercised.

Until the release asset is replaced, the download there is the 03:28 build (554,727,395 bytes, SHA-256 `3C20FE37E014A76A9D1B268FFAED1B8CE646AB1447A0C035674CE20D661C52DD`), which predates the marketplace fixes. The values above were measured at 14:38 on `dist-exe\DeepSeek-Harness-Setup.exe` with both commands shown, and that is the file to attach.
