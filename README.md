# Ungoogled Chromium Portable Builder

[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

[简体中文](README.zh-CN.md)

Automatically builds portable [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium-windows) packages with the [Chrome++](https://github.com/Bush2021/chrome_plus) patch integrated. Supports x64 / x86 / arm64, and GitHub Actions checks upstream updates every 12 hours.

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

## GitHub Actions

[build-portable.yml](.github/workflows/build-portable.yml) supports:

- **Manual runs**: choose an architecture and optionally publish a Release
- **Scheduled runs**: check upstream every 12 hours (00:00 / 12:00 UTC) and build all three architectures in parallel

Release tag format: `<ungoogled_tag>-<chrome_plus_tag>` (for example, `147.0.7727.116-1.1-1.16.0`). Existing tags are skipped automatically.

**Self-hosted runner**: for manual runs, enter `["self-hosted","Windows","X64"]`; for scheduled runs, set the repository variable `PORTABLE_RUNNER_LABELS`.

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
