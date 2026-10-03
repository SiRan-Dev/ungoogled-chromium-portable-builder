# Ungoogled Chromium Portable Builder

[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

[简体中文](README.zh-CN.md)

Automatically builds portable [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium-windows) packages with the [Chrome++](https://github.com/BlackRosePetals/chromeplus) patch integrated. Supports x64 / x86 / arm64, and GitHub Actions checks upstream updates every 12 hours.

## Upstream sources

| Component | Repository |
|-----------|------------|
| ungoogled-chromium | [`ungoogled-software/ungoogled-chromium-windows`](https://github.com/ungoogled-software/ungoogled-chromium-windows) |
| Chrome++ | [`BlackRosePetals/chromeplus`](https://github.com/BlackRosePetals/chromeplus) |

The original `Bush2021/chrome_plus` repository is no longer available, so the build uses the maintained fork above. It ships the same asset layout (`<arch>/App/version.dll` and `<arch>/App/chrome++.ini`), so no other change is required. If either upstream moves again, point the build at the new repository with `-ChromePlusRepo` / `-UngoogledRepo` instead of editing the script.


## Pipeline

```text
installer.exe -> 7-Zip extraction -> browser directory -> inject Chrome++ -> portable artifact
```

## Artifact Layout

```text
artifacts/ungoogled-chromium-portable-<arch>/
|-- App/           # Browser + Chrome++ patch
|-- Data/          # User data
|-- Cache/         # Cache
`-- metadata.json
```

The default Chrome++ configuration already points to `Data` / `Cache`, so the build can run portably without additional changes.

## Quick Start

**Requirements**: PowerShell 7+, `7z.exe` in `PATH`, access to the GitHub API

```powershell
# Default: x64
pwsh .\build-portable.ps1

# Specify architecture and overwrite existing artifacts
pwsh .\build-portable.ps1 -Arch arm64 -Force
```

| Parameter | Description |
|-----------|-------------|
| `-Arch` | `x64` / `x86` / `arm64`, defaults to `x64` |
| `-Force` | Overwrite existing artifacts |
| `-SkipArchive` | Generate the directory only, without creating a `.7z` archive |
| `-ChromePlusIniPath` | Use a custom `chrome++.ini` |
| `-KeepWorkDir` | Keep the temporary extraction directory |
| `-UngoogledRepo` | Upstream ungoogled-chromium repository, defaults to `ungoogled-software/ungoogled-chromium-windows` |
| `-ChromePlusRepo` | Upstream Chrome++ repository, defaults to `BlackRosePetals/chromeplus` |
| `-ResolveOnly` | Only resolve the latest upstream versions and print the release tag, without downloading anything |

## GitHub Actions

[build-portable.yml](.github/workflows/build-portable.yml) supports:

- **Manual runs**: choose an architecture and optionally publish a Release
- **Scheduled runs**: check upstream every 12 hours (00:00 / 12:00 UTC) and build all three architectures in parallel

Release tag format: `<ungoogled_tag>-<chrome_plus_version>` (for example, `147.0.7727.116-1.1-1.16.0`). Existing tags are skipped automatically, so a scheduled run only builds when upstream actually published something new.

### Manual run inputs

| Input | Description |
|-------|-------------|
| `arch` | `all` (default) builds x64, x86 and arm64 in parallel |
| `publish` | Create a GitHub Release for the built packages |
| `force` | Build even if the release tag already exists |
| `runner_labels` | Runner labels as a JSON array, e.g. `["self-hosted","Windows","X64"]` |

### Repository variables

| Variable | Description |
|----------|-------------|
| `PORTABLE_RUNNER_LABELS` | Runner labels used by scheduled runs, as a JSON array. Defaults to `["self-hosted","Windows","X64"]`. Use `["windows-latest"]` to build on GitHub-hosted runners instead. |
| `DEFENDER_EXCLUSION` | Optional. A path added to the Microsoft Defender exclusion list before building, for example `D:\actions\ungoogled-chromium-portable-builder`. Leave empty to skip. |

### How a run works

1. `detect` resolves the latest upstream versions with `build-portable.ps1 -ResolveOnly` and computes the release tag.
2. `detect` compares that tag against the existing tags. If it already exists, the run finishes without building.
3. `build` runs once per architecture, packing `artifacts/ungoogled-chromium-portable-<arch>.7z` and uploading it as a workflow artifact.
4. `release` downloads the packages and creates the GitHub Release, replacing the assets if the release already exists.

## Ungoogled Chromium Notes

Ungoogled Chromium uses Google Search by default in Chromium, but the Ungoogled build removes Google services during de-Googling. This also removes the built-in Google search engine entry. After first launch, open Settings and manually choose another search engine, or add a custom Google search engine yourself.

Ungoogled Chromium also does not support installing extensions directly from the Chrome Web Store by default. You can enable it with the following steps:

1. Open `chrome://flags/#extension-mime-request-handling`.
2. Set the flag to `Always prompt for install`. If it is already set to this option, skip this step.
3. Install [Chromium Web Store](https://github.com/NeverDecaf/chromium-web-store/releases/). Use a proxy if the link cannot be opened from your network.
4. After the extension is installed, you can install other extensions online from the Chrome Web Store.

## Notes

- Chrome++ `version.dll` may be blocked by Windows Defender. Add an exclusion if needed.
- The same may happen on GitHub-hosted runners. A self-hosted runner with the proper exclusion configured is recommended.

## License

[GNU General Public License v3.0](LICENSE) - Copyright (c) 2026, ReRanr
