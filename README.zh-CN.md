# Ungoogled Chromium Portable Builder

[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

[English](README.md)

自动构建 [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium-windows) 便携版，集成 [Chrome++](https://github.com/Bush2021/chrome_plus) 补丁。支持 x64 / x86 / arm64，GitHub Actions 每 12 小时自动检查上游更新。

## 上游来源

| 组件 | 仓库 |
|------|------|
| ungoogled-chromium | [`ungoogled-software/ungoogled-chromium-windows`](https://github.com/ungoogled-software/ungoogled-chromium-windows) |
| Chrome++ | [`Bush2021/chrome_plus`](https://github.com/Bush2021/chrome_plus) |

Chrome++ 的压缩包结构为 `<arch>/App/version.dll` 与 `<arch>/App/chrome++.ini`。如果上游迁移，用 `-ChromePlusRepo` / `-UngoogledRepo` 指定新仓库即可，不必改脚本。

## 流程

```text
installer.exe -> 7-Zip 解压 -> 浏览器目录 -> 注入 Chrome++ -> 便携版产物
```

## 产物结构

```text
artifacts/ungoogled-chromium-portable-<arch>/
|-- App/           # 浏览器 + Chrome++ 补丁
|-- Data/          # 用户数据
|-- Cache/         # 缓存
`-- metadata.json
```

Chrome++ 默认配置已指向 `Data` / `Cache`，无需修改即可便携运行。

## 快速开始

**依赖**：PowerShell 7+、`7z.exe` 在 PATH 中、可访问 GitHub API

```powershell
# 默认 x64
pwsh .\build-portable.ps1

# 指定架构，强制覆盖
pwsh .\build-portable.ps1 -Arch arm64 -Force
```

| 参数 | 说明 |
|------|------|
| `-Arch` | `x64` / `x86` / `arm64`，默认 `x64` |
| `-Force` | 覆盖已有产物 |
| `-SkipArchive` | 只生成目录，不打 `.7z` |
| `-ChromePlusIniPath` | 自定义 `chrome++.ini` |
| `-KeepWorkDir` | 保留解压临时目录 |
| `-UngoogledRepo` | ungoogled-chromium 上游仓库，默认 `ungoogled-software/ungoogled-chromium-windows` |
| `-ChromePlusRepo` | Chrome++ 上游仓库，默认 `Bush2021/chrome_plus` |
| `-ResolveOnly` | 只解析上游最新版本并输出发布 Tag，不下载任何文件 |

## GitHub Actions

[build-portable.yml](.github/workflows/build-portable.yml) 支持：

- **手动触发**：选择架构，可选发布 Release
- **定时任务**：每 12 小时（00:00 / 12:00 UTC）检查上游，三架构并行构建

标签格式：`<ungoogled_tag>-<chrome_plus_version>`（如 `147.0.7727.116-1.1-1.16.0`）。已存在则自动跳过，所以定时任务只在上游真的发布了新版本时才会构建。

### 手动触发参数

| 参数 | 说明 |
|------|------|
| `arch` | `all`（默认）并行构建 x64、x86、arm64 |
| `publish` | 为构建产物创建 GitHub Release |
| `force` | 即使 Release 标签已存在也强制重新构建 |
| `runner_labels` | Runner 标签，JSON 数组格式，如 `["self-hosted","Windows","X64"]` |

### 仓库变量

| 变量 | 说明 |
|------|------|
| `PORTABLE_RUNNER_LABELS` | 定时任务使用的 Runner 标签，JSON 数组格式，默认 `["self-hosted","Windows","X64"]`。改成 `["windows-latest"]` 即使用 GitHub 官方 Runner。 |
| `DEFENDER_EXCLUSION` | 可选。构建前加入 Microsoft Defender 排除项的路径，例如 `D:\actions\ungoogled-chromium-portable-builder`。留空则跳过。 |

### 运行流程

1. `detect` 用 `build-portable.ps1 -ResolveOnly` 解析上游最新版本并算出发布 Tag。
2. `detect` 把该 Tag 与现有 Tag 比对，若已存在则直接结束，不做任何构建。
3. `build` 按架构并行执行，产出 `artifacts/ungoogled-chromium-portable-<arch>.7z` 并作为工作流产物上传。
4. `release` 下载产物并创建 GitHub Release；若 Release 已存在则替换其中的附件。

## Ungoogled Chromium 注意事项

Ungoogled Chromium 基于 Chromium，而 Chromium 默认使用 Google 搜索引擎；但 Ungoogled Chromium 在去谷歌化时也一并移除了 Google 搜索引擎条目。因此首次使用时需要在设置中手动选择其他搜索引擎，或者自定义添加一个 Google 搜索引擎。

Ungoogled Chromium 默认不支持直接通过 Chrome 网上应用店安装扩展，但可以通过下面的方法解决：

1. 打开 `chrome://flags/#extension-mime-request-handling`。
2. 将选项设置为 `Always prompt for install`。如果已经是这个选项，就跳过这一步。
3. 安装 [Chromium Web Store](https://github.com/NeverDecaf/chromium-web-store/releases/)。如果链接打不开，请自行使用代理。
4. 装好这个扩展以后，就可以在线安装其他插件了。

## 注意事项

- `Chrome++` 的 `version.dll` 可能被 Windows Defender 拦截，需添加排除项
- GitHub-hosted Runner 同理，建议用自托管 runner 配置排除

## 许可证

[GNU General Public License v3.0](LICENSE) - Copyright (c) 2026, EchoRan
