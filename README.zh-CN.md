# Ungoogled Chromium Portable Builder

[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

[English](README.md)

自动构建 [ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium-windows) 便携版，集成 [Chrome++](https://github.com/Bush2021/chrome_plus) 补丁。支持 x64 / x86 / arm64，GitHub Actions 每 12 小时自动检查上游更新。

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

## GitHub Actions

[build-portable.yml](.github/workflows/build-portable.yml) 支持：

- **手动触发**：选择架构，可选发布 Release
- **定时任务**：每 12 小时（00:00 / 12:00 UTC）检查上游，三架构并行构建

标签格式：`<ungoogled_tag>-<chrome_plus_tag>`（如 `147.0.7727.116-1.1-1.16.0`），已存在则自动跳过。

**自托管 Runner**：手动触发时输入 `["self-hosted","Windows","X64"]`；定时任务设置仓库变量 `PORTABLE_RUNNER_LABELS`。

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
