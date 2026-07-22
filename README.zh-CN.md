# Vibe-Trading Desktop Community 0.3.0

[English](README.en.md) | [简要说明](README.md)

## 项目介绍

Vibe-Trading Desktop Community 是为 [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) 制作的非官方 Windows 桌面端原型。它保留原项目的 React 界面和 Python/FastAPI Agent，在外层增加桌面窗口、Windows 安装包、本地进程管理，以及面向桌面使用场景的安全和体验优化。

- 上游包版本基线：Vibe-Trading `0.1.11`
- 桌面端版本：`0.3.0 Alpha`
- 支持平台：Windows 10/11 x64
- 技术结构：Electron + React/Vite + FastAPI + 内置 Python 3.12 运行环境

这是社区 Alpha，不是 HKUDS 官方桌面客户端。它仅供研究和评估，不构成投资建议。

![Vibe-Trading Desktop 首页](preview/desktop-home.png)

## 已实现的内容

### 桌面宿主和安装包

- 双击桌面应用后自动启动现有 Vibe-Trading 本地后端，并在独立窗口中加载界面。
- 每次启动使用随机 `127.0.0.1` 端口和临时认证密钥。
- 后端健康检查通过后才展示应用界面。
- 支持单实例、启动诊断、日志记录和受控退出。
- 应用退出时清理内置 Python 子进程，避免后台残留。
- Windows 安装包内含 Electron 应用、前端资源和隔离的 Python 运行环境。
- 使用上游项目图标，并保留许可证与声明文件。

### 凭据与本地安全

- 桌面端 API 凭据迁移到 Electron `safeStorage`，在 Windows 上使用当前用户级系统加密。
- 凭据在启动后端时注入，不在界面或常规日志中显示明文。
- 首次启动会说明本地数据、凭据处理、非官方身份和金融风险。
- 后端默认只监听本机回环地址，不向局域网开放。

### 模型和聊天体验

- 用户填写有效 Key 后，可从所选服务商加载模型。
- 下拉框显示完整模型列表，不再被当前输入内容过滤掉其他模型。
- 模型选择器与整体界面风格统一，同时保留手动填写模型名的能力。
- 聊天界面显示实际配置的服务商、模型和推理强度。
- API 返回真实模型字段时会记录，并显示每次回复耗时。
- 修复回复右上角复制按钮，同时保留框选文本和 `Ctrl+C`。

模型在自然语言中自称什么不能证明实际调用了哪个模型，因此桌面端以真实配置和 API 返回元数据为准。

### 页面加载与导航

- 启动后预加载主要页面代码，降低第一次进入各页面时的等待感。
- 继续通过本地 FastAPI 以同源方式提供页面，而不是使用 `file://`，从而保留 REST、SSE、文件上传和 SPA 路由行为。

### IM 通道中心

- 读取上游通道注册表，不把界面限制为微信或某一个平台。
- 展示已有适配器、依赖状态、配置状态、运行状态和配对提示。
- 保留微信二维码/配对流程；其他上游支持的平台在安装可选依赖并提供相应账号或凭据后也可使用。

### 发布和更新基础

- 提供 Windows 构建与 Release 资产的 GitHub Actions 草案。
- 提供基于 GitHub Releases 的手动检查更新基础能力。
- 当前本地 `0.3.0` 未绑定稳定发布仓库；在真实升级链路验证完成前，更新源保持禁用。

## 技术结构

```text
Windows 桌面应用
        |
        +-- Electron 主进程
        |     +-- 窗口和应用生命周期
        |     +-- 系统加密凭据存储
        |     +-- 后端健康检查与日志
        |     +-- 检查更新基础能力
        |
        +-- 内置 Python 运行环境
              +-- 127.0.0.1:<随机端口> 上的本地 FastAPI 服务
              +-- 原有 Agent、工具、数据源和连接器
              +-- 托管编译后的 React 页面
```

前后端保持在同一本地来源，可以继续复用原项目的 REST、SSE、上传和路由机制。

## 安装方法

公共 Git 仓库中**不包含 EXE 安装程序**。Git 中保存的是源码和校验信息；生成的安装程序必须单独作为 GitHub Release 附件发布。

1. 打开本仓库的 [Releases 页面](https://github.com/QCYTSN/vibe-trading-desktop-handoff/releases)。
2. Release 发布后，下载 `Vibe-Trading-Desktop-Community-0.3.0-x64.exe` 和对应校验文件。
3. 在 PowerShell 中核对 SHA-256：

   ```powershell
   Get-FileHash .\Vibe-Trading-Desktop-Community-0.3.0-x64.exe -Algorithm SHA256
   ```

4. 运行安装程序。当前版本尚未签名，Windows 可能显示 SmartScreen 提示。
5. 阅读首次启动提示，然后在“设置”中填写你自己的模型服务商和 API 凭据。

不要把 API Key 发到 Issue、日志、截图或仓库文件中。

## 验证状态

已在开发机完成：

- 前端测试 282/282 通过。
- 相关后端设置、通道、QVeris 和运行元数据测试 57/57 通过。
- 内置后端烟雾测试通过。
- 接口烟雾测试覆盖 68 个 OpenAPI 操作，以及首页、静态资源、SSE ticket 和配置路径。
- 解包版桌面应用可正常启动、加载页面、退出，并且退出后没有打包版 Electron 或 Python 进程残留。
- 使用真实 API 手动验证了服务商/模型加载、聊天回复、工具调用过程、详细结果和任务正常结束。
- 较早的安装包构建已在干净 Windows 11 VirtualBox 环境中成功安装和启动。

在称为正式可发布版本前仍需完成：

- 用最终 `0.3.0` 安装包在干净 Windows 快照中重新执行完整回归。
- 发布测试版 `0.3.0 -> 0.3.1`，验证检查、下载、校验、重启、升级和回滚行为。
- 使用真实账号和各平台专用 SDK/凭据验证可选 IM 通道。
- 完成依赖许可证、内置二进制文件和 SBOM 的最终审计。

## 已知缺陷和后续工作

- **安装包未签名：** 在项目负责人确定代码签名方案前，SmartScreen 提示属于预期现象。
- **体积较大：** 当前安装包约 316 MiB，解包后约 1.17 GiB；主要优化对象是 Python 科学计算和连接器依赖。
- **冷启动路径：** 路由预加载改善了重复访问，但冷启动和首次使用耗时仍需继续分析和分阶段加载。
- **自动更新：** 基础代码已存在，但在发布仓库、签名和真实升级测试确定前不会启用线上更新。
- **可选集成：** 部分连接器和 IM 通道必须安装额外 SDK，并需要各平台账号、Key 或地区可用性。
- **目前仅支持 Windows：** macOS 和 Linux 尚未制作安装包。
- **源码来源限制：** 当前工作基线来自没有 `.git` 历史的源码快照。`source-overlay` 保存了改动后的相关文件，但不是可直接合并的上游提交序列。

## 仓库结构

```text
.
├── README.md
├── README.en.md
├── README.zh-CN.md
├── LICENSE
├── NOTICE
├── preview/
│   ├── desktop-home.png
│   └── desktop-onboarding.png
├── release-assets/
│   └── SHA256SUMS-0.3.0.txt
└── source-overlay/
    ├── .github/       # Windows Release 工作流草案
    ├── agent/         # 后端新增和修改
    ├── assets/        # 桌面资源和声明
    ├── desktop/       # Electron 宿主与打包
    └── frontend/      # 桌面体验相关前端修改
```

安装程序不进入普通 Git 历史，应作为 GitHub Release 附件发布。运行环境、依赖目录和构建输出等可生成内容也不会提交。

## 开发和构建说明

> **构建前提：** 当前公共仓库只是 overlay，并不是完整可构建的 Vibe-Trading 源码树。执行任何构建命令前，必须先把这些文件还原到与 `0.1.11` 基线匹配的完整上游 Vibe-Trading 源码中；不要直接在这个独立 overlay 仓库根目录构建。

还原 overlay 后，还需要准备：

- React 和 Electron 项目所需的 Node.js/Bun 依赖
- Python 3.12 和 Vibe-Trading 后端依赖
- 编译并放到后端静态托管位置的前端资源
- 用于 Windows 分发的隔离 Python 运行环境

如果后续要向上游贡献，应从可确认提交的官方仓库 fork 重新整理这些改动，并拆成便于审查的提交。不要把当前快照 overlay 当成可直接合并的 Pull Request。

## 安全与金融风险

- 只使用你本人控制的 API Key 和交易账户。
- 优先使用可撤销、权限受限的凭据，并只授予必要权限。
- 任何真实交易前，都要自行复核策略、市场数据、额度和连接器行为。
- 模型输出可能错误、不完整、过时，或不适用于特定市场。
- 本应用名称和仓库不代表 HKUDS 或任何模型、数据服务商的认可。

## 许可证

项目继续遵循上游 MIT License。Vibe-Trading 及其原始成果归原作者所有；重新分发的第三方组件继续受各自许可证和声明约束。
