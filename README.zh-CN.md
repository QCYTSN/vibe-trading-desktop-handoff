# Vibe-Trading Desktop Community 0.3.0 交接说明

更新时间：2026-07-21

## 1. 这是什么

这是基于 [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) 的 Windows 社区桌面化原型。它没有重写 Vibe-Trading，而是在原有 React + FastAPI + Python Agent 外增加 Electron 桌面外壳、内置运行时、安装器、安全存储、更新流程和少量必要的前后端接口。

当前名称为 **Vibe-Trading Desktop Community**。在 HKUDS 明确认可之前，它不是官方客户端，不应使用“Official”字样。

版本关系：

- 上游源码基线：Vibe-Trading `0.1.11`
- 桌面层版本：`0.3.0`
- 当前目标系统：Windows 10/11 x64
- 安装器格式：Electron + NSIS `.exe`
- 后端运行时：内置 Python 3.12 目录运行时，不使用 PyInstaller `--onefile`

重要：原始工作目录是源码 ZIP，没有 `.git` 历史。本交接包是一个按原路径保存的 **source overlay**，不是可以直接提交的 Git patch。上游 `main` 在 0.1.11 发布后仍持续更新，因此合并到最新 `main` 时必须逐项重放和解决冲突，不应无检查地覆盖最新代码。

## 2. 目录内容

```text
Vibe-Trading-Desktop-Handoff-0.3.0/
├── README.zh-CN.md               # 本文
├── README.en.md                  # 英文交接说明和英文联系模板
├── release-assets/               # 本地 Release 上传暂存目录
│   ├── Vibe-Trading-Desktop-Community-0.3.0-x64.exe
│   └── SHA256SUMS-0.3.0.txt
├── preview/
│   └── desktop-0.3.0-window.png  # 已验证桌面窗口预览
└── source-overlay/               # 按上游相对路径保存的新增/修改源码
│   ├── .github/workflows/
│   ├── desktop/electron/
│   ├── frontend/src/
│   ├── agent/src/
│   ├── agent/tests/
│   ├── assets/icon.png
│   ├── LICENSE
│   └── NOTICE
```

为保持交接包轻量，以下内容没有复制：

- `node_modules`
- 内置 Python `runtime/backend`
- Electron/Python 下载缓存和构建日志
- `frontend/dist`、TypeScript `dist`
- `release/win-unpacked`

这些都是可由源码和工作流重新生成的产物。本地工作副本会把安装器暂存在 `release-assets/`，但 `.gitignore` 会阻止约 316 MiB 的 EXE 进入 Git 历史；实际体验版本应上传为 GitHub Release 资产。

```text
release-assets\Vibe-Trading-Desktop-Community-0.3.0-x64.exe
```

SHA-256：

```text
c27dfd2408c5b1218c948a7f775e948b9885c548a072ce6f426fa6099f88e3d1
```

当前安装器约 316 MiB，解包应用约 1.17 GiB，尚未签名，只适合内部验收或明确标注的 Alpha 测试。

### 给维护者的最短体验步骤

1. 使用 Windows 10/11 x64。
2. 从 GitHub Release 下载安装器，并核对 `release-assets/SHA256SUMS-0.3.0.txt`。
3. 双击安装器；Windows SmartScreen 出现警告是因为当前 Alpha 尚未签名，不代表已经获得官方信任。
4. 安装完成后启动 **Vibe-Trading Desktop Community**。
5. 阅读并接受首次启动说明。
6. 在 Settings 中配置自己的模型供应商、模型和 API Key；交接包不包含任何用户密钥。
7. 测试聊天、模型显示、回复耗时、复制按钮、主要页面和 IM Channel Center。
8. 关闭窗口后确认桌面端及其内置 Python 后端一同退出。

源码审查直接使用本仓库；实际安装体验使用 GitHub Release 中的安装器。

## 3. 新增和优化内容

### 3.1 桌面宿主与安装器

- Electron 主进程启动内置 FastAPI/Python 后端。
- 每次启动自动选择随机的 `127.0.0.1` 端口。
- 每次启动生成独立的随机 `API_AUTH_KEY`，不使用固定本地口令。
- 轮询 `/health` 成功后才加载原有 Web UI，并提供加载/失败页面。
- 单实例锁，避免重复打开多个后端。
- 应用关闭时先请求正常停止，再清理所拥有的后端进程树。
- 渲染进程禁用 Node integration，使用 sandbox 和受限 preload IPC。
- Windows NSIS 安装器、桌面/开始菜单快捷方式、上游项目图标。
- 内置 Python 3.12、前端静态文件及 PDF/WeasyPrint 所需的最小 GTK/Pango 运行库。
- 保留 MIT `LICENSE`、`NOTICE` 和桌面隐私/安全/发布说明。

### 3.2 凭据与本地安全

- 使用 Electron `safeStorage`，在 Windows 上由 DPAPI 加密保存支持的密钥。
- 支持迁移 LLM、Tushare、QVeris、个人微信及 IM 通道顶层敏感字段。
- 密钥通过子进程环境变量注入后端，不在桌面日志中打印明文。
- 首次启动增加社区非官方说明、本地数据说明、模型局限及真实交易风险确认。

这不是完整的安全审计。公开发布前仍需要重新审查所有可选连接器的密钥字段、日志脱敏、依赖供应链和更新签名。

### 3.3 模型和聊天体验

- 模型选择器支持使用已配置 Key 动态加载供应商模型列表。
- 下拉列表会展示全部已发现模型，不再要求先删除手填文字才能看到其他模型。
- 模型下拉样式和手感与设置页其他控件统一，同时保留手动输入自定义模型的能力。
- 聊天界面固定显示真实运行配置，例如 `DeepSeek · deepseek-v4-flash` 和推理强度。
- 每次回复显示耗时。
- 后端捕获并持久化 API 返回的真实 `model` 元数据，不再依靠模型自己回答“你是什么模型”。
- 修复桌面窗口中回复右上角复制按钮失效的问题。
- 增加温度和推理强度的说明，不改变金融 Agent 的核心系统提示词。

### 3.4 页面加载和前端体验

- 对主要路由模块进行预加载，改善第一次打开页面的等待感。
- 保持 FastAPI 同源托管 React，避免 `file://`、CORS、SSE 和 SPA 深层路由问题。
- 首次启动门、桌面更新设置卡片和桌面环境类型声明。
- 本地字体/资源及页面加载细节优化。

### 3.5 IM 通道中心

- 新增由后端注册表驱动的桌面通道中心，不只支持微信。
- 展示上游内置的 WebSocket、Telegram、Slack、Discord、Matrix、WhatsApp、Signal、QQ/NapCat、微信、企业微信、飞书、钉钉、Teams、Email、Mochat 等适配器。
- 展示可用性、缺失依赖、配置字段、启用状态、运行状态和恢复提示。
- 支持启动、停止、配对命令；个人微信提供专门的二维码登录流程。
- 不会假装所有通道开箱即用：需要对应 SDK、Bot/企业账号或平台凭据的通道仍需按平台配置。

### 3.6 更新和发布流水线

- 设置页提供手动“检查更新”。
- 支持下载进度、下载完成后重启安装。
- GitHub Releases 使用 `latest.yml`、安装器和 `.blockmap`；blockmap 可支持差分下载。
- GitHub Actions Windows 工作流构建前端和内置后端、运行烟雾测试、打包、计算 SHA-256，并创建草稿 Release。
- 工作流目前接受传统证书 secrets：`CSC_LINK` 和 `CSC_KEY_PASSWORD`。
- 本地 0.3.0 构建未绑定一个真实发布仓库，因此应用内更新功能保持禁用；选定 fork/官方仓库后，CI 会写入正确的 release repository。

永远不要用相同版本号替换已经发布的二进制文件。出现问题应发布更高版本。

## 4. 已完成验证

- 前端：282/282 测试通过。
- 相关后端：57/57 设置、通道、QVeris、运行时元数据测试通过。
- 内置后端启动烟雾测试通过，健康检查约 6.6 秒内完成。
- 接口烟雾测试覆盖 68 个 OpenAPI 操作以及主页面、静态资源、SSE ticket 等关键路径。
- Windows 解包应用成功启动内置 Python 后端并显示首次启动界面。
- 关闭窗口后，Electron/Python 残留进程数为零。
- 较早桌面构建已在干净 Windows 11 VirtualBox 虚拟机完成安装和主要页面/LLM/工具调用验收。

尚未完成：

- 最终 0.3.0 在全新 Windows 11 快照上的重新安装验收。
- 真实 GitHub 仓库中的 `N -> N+1` 自动更新全链路测试。
- 受信任 Authenticode 签名及 SmartScreen 实机观察。
- 所有 16 个 IM 平台的真实账号端到端测试。
- 完整第三方许可证/SBOM 审计。
- 安装器体积和首次页面冷加载的第二轮专项优化。

本机完整 Python 测试套件曾被 Anaconda 环境中的 NumPy 2.3.5 与旧 pandas/pyarrow ABI 冲突阻塞；这不是已定位到本项目改动的代码失败。公开 PR 前应在隔离、锁定依赖的环境或官方 CI 上重新运行完整套件。

## 5. 如何在真正的 Git 仓库中继续开发

推荐流程：

```powershell
git clone https://github.com/HKUDS/Vibe-Trading.git
cd Vibe-Trading
git switch -c feat/windows-desktop-shell
```

然后不要一次性盲目覆盖最新 `main`。建议按下列顺序从 `source-overlay` 重放：

1. `desktop/electron/` 独立桌面层；
2. `.github/workflows/desktop-windows.yml`；
3. 最小后端 API/运行时元数据改动；
4. 前端设置、模型、聊天和通道 UI；
5. 测试和文档。

每一块单独提交并运行相应测试。上游要求社区 PR 的每个 commit 都带 DCO `Signed-off-by:`，例如：

```powershell
git commit -s -m "feat(desktop): add Windows Electron shell"
```

不要在 commit 或 PR 描述中加入 AI `Co-Authored-By`；上游明确要求只保留 DCO sign-off。

## 6. 本地构建和发布

在仓库根目录先构建前端，然后进入桌面目录：

```powershell
cd frontend
npm ci
npm run build

cd ..\desktop\electron
npm ci
npm run runtime:win
npm run smoke:backend
npm run smoke:interfaces
npm run pack:win
npm run installer:win
npm run verify:release
```

常用产物：

- `release/win-unpacked/`：无需安装的调试版。
- `release/Vibe-Trading-Desktop-Community-<version>-x64.exe`：NSIS 安装器。
- `release/latest.yml`：electron-updater 元数据。
- `release/*.blockmap`：差分更新元数据。
- `release/SHA256SUMS.txt`：发布校验值。

## 7. Windows 代码签名证书怎么做

### 7.1 先决定签名归属

最推荐的顺序：

1. 先在上游 Discussions 的 Ideas 分类联系维护者。
2. 确认项目是并入主仓库、单独官方仓库，还是社区 fork。
3. 再决定应用名称、GitHub Releases 更新源和签名主体。

如果上游接收为官方桌面端，应由 HKUDS/其认可的法律主体持有签名身份、签名服务和 GitHub secrets。贡献者不应该把私人证书或 PFX 交给他人，也不应把证书放进 Git。

### 7.2 可选方案

**A. SignPath Foundation（开源项目优先尝试）**

- 对符合条件的开源项目提供免费托管签名。
- 适合没有公司主体、希望签名留在受控 CI 流水线的社区项目。
- 需要项目公开、申请审核并按其流水线要求配置。
- 官方入口：https://signpath.io/solutions/open-source-community

**B. 商业 OV 代码签名证书**

- 向 Microsoft Trusted Root Program 中的 CA 购买，例如 DigiCert、Sectigo、GlobalSign 等。
- CA 会验证个人或组织身份。
- 现代 OV 私钥通常需要硬件 token 或云 HSM，不能把可导出的私钥随意塞入 GitHub secrets。
- 对位于中国大陆的个人/组织，这是比 Azure Artifact Signing 更现实的传统路线，但购买前要向 CA 确认其当前地区、个人/组织验证和 CI/HSM支持。

**C. Azure Artifact Signing（原 Trusted Signing）**

- 微软推荐用于 Store 外发布，并可直接集成 CI。
- 但微软当前公开说明中，组织仅限美国、加拿大、欧盟、英国；个人仅限美国和加拿大。以中国大陆个人身份目前不适合作为首选。
- 如果未来由符合地区条件的上游组织持有，可在 electron-builder 中改用 `win.azureSignOptions` 和 Azure 身份环境变量。

**D. Microsoft Store 的 MSIX**

- Store 提交的 MSIX 由 Microsoft 免费重新签名。
- 当前工程产物是 NSIS EXE，不是 MSIX；转换还要验证 Python sidecar、文件写入、回环服务和 Store policy，不是简单换扩展名。

**E. 自签名证书**

- 只适合自己电脑、虚拟机或由单位统一下发根证书的内部环境。
- 对公网用户不会建立系统信任，也不能解决 SmartScreen，因此不应作为公开发布方案。

### 7.3 当前工程需要怎样接入

传统可导出 PFX 的最小接入方式：

1. 在私有 CI secret 中配置 `CSC_LINK`（证书文件的 base64/data URL 或受保护路径）和 `CSC_KEY_PASSWORD`。
2. GitHub Actions 运行 electron-builder 时自动签署内部应用 EXE 和 NSIS 安装器。
3. 运行 `Get-AuthenticodeSignature` 和 SignTool 校验。
4. 必须使用 SHA-256 和 RFC 3161 时间戳，保证证书到期后已签名版本仍可验证。

验证示例：

```powershell
Get-AuthenticodeSignature .\Vibe-Trading-Desktop-Community-0.3.0-x64.exe
signtool verify /pa /all /v .\Vibe-Trading-Desktop-Community-0.3.0-x64.exe
```

公开稳定版应显示 `Valid`。注意：可信签名确认发布者和文件未被篡改，但新程序仍可能需要一段时间积累 SmartScreen reputation；现在不要为了“立刻无警告”盲目购买昂贵 EV 证书。

## 8. 如何联系原作者

仓库没有在 README 或贡献指南中公开项目邮箱。当前最合适、最透明的入口是：

1. [GitHub Discussions](https://github.com/HKUDS/Vibe-Trading/discussions) 的 **Ideas** 分类；
2. 如果数日没有回应，再按贡献指南使用 [Feature request](https://github.com/HKUDS/Vibe-Trading/issues/new/choose)；
3. 讨论取得方向后，再从真正的 fork 创建分块、带 DCO 的 Draft PR。

项目 Discussions 已启用；`@warren618`（Haozhe Wu）在仓库公告、讨论和维护活动中非常活跃，可在讨论中礼貌 @ 他。不要先私下发送一个来源不明的 EXE，也不要先宣称这是官方桌面端。最有效的方式是公开说明架构、验证、已知限制和你希望维护者决定的问题。

建议 Discussion 标题：

```text
Proposal: Windows desktop shell and installer for Vibe-Trading
```

建议发送的英文正文已经完整写在 `README.en.md` 的“Maintainer contact template”章节。首次联系时先给源码分支/交接说明和截图；安装器可作为可选测试资产，同时附 SHA-256，并明确“unsigned Alpha / unofficial”。

希望维护者明确回答：

1. 代码应进入主仓库的 `desktop/`、单独 companion repo，还是保持 community fork？
2. 审核阶段允许使用什么名称、图标和商标措辞？
3. 如果官方化，谁持有 Windows 签名和 GitHub Releases 更新源？
4. 默认 Windows 运行时应内置哪些可选 IM SDK？
5. 个人微信 iLink 是否适合作为官方桌面分发支持面？
6. 维护者希望先审桌面外壳，还是先审最小后端/前端接口？

## 9. 下一阶段优先级

1. 在真实 fork 上重建可审查的 DCO commit 历史，并同步最新 `main`。
2. 联系上游，确定仓库、品牌、更新源和签名归属。
3. 将最终 0.3.0 安装到干净 Windows 11 快照，完成回归。
4. 在测试仓库发布 `0.3.0 -> 0.3.1`，验证检查、下载、重启和升级。
5. 申请 SignPath 或由确定的发布主体采购/配置证书。
6. 做依赖体积分析，按“核心运行时 + 可选连接器包”拆分，而不是随意删除 Python 包。
7. 对首次路由加载、后端冷启动和杀毒软件扫描耗时分别打点后再优化。
8. 生成第三方许可证清单和 SBOM，补齐正式发布说明。

## 10. 许可证与责任

上游使用 MIT License。本交接包保留了 `LICENSE` 和 `NOTICE`。公开发布时必须保留上游版权和许可信息，并继续使用“Community/Unofficial”措辞，直到 HKUDS 书面或公开确认官方身份。

这是金融研究工具，不保证模型、数据或策略正确，也不应绕过上游现有的真实交易授权、风控、确认和审计边界。
