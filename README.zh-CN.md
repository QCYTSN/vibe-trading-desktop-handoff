# Vibe-Trading Desktop Community — 历史交接仓库

[English](README.en.md) | [简要说明](README.md)

## 当前状态

这个仓库保存的是最初用于向原作者展示 Windows 桌面端原型的
“源码优先交接材料”，现在已经不再承担实际开发。

相关工作已经从真实 fork 的最新上游源码重新构建，并按维护者要求拆成了
可独立审查的 Pull Request：

| Pull Request | 内容 | 状态 |
| --- | --- | --- |
| [#923](https://github.com/HKUDS/Vibe-Trading/pull/923) | Electron 桌面外壳、回环鉴权、诊断、进程生命周期、看门狗、五语言 | 已合并 |
| [#924](https://github.com/HKUDS/Vibe-Trading/pull/924) | 服务商/模型发现、模型选择、真实运行配置与耗时、复制修复 | 已合并 |
| [#1015](https://github.com/HKUDS/Vibe-Trading/pull/1015) | Windows NSIS 打包、隔离 Python 3.12、Electron safeStorage | Draft 审查中 |

现在的源码唯一可信来源是
[HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) 和
[QCYTSN 的 fork](https://github.com/QCYTSN/Vibe-Trading)。

## 为什么归档这个仓库

`source-overlay/` 来源于较早的 0.1.11 无 Git 历史源码快照。它当时适合展示
原型，但无法提供干净的上游合并基线，现在也已经过时。继续在 overlay 上
开发会重新引入已经解决的同步和审查问题。

因此，这个仓库只保留提案来源、截图和早期验证记录，不再是：

- 当前 Vibe-Trading 源码；
- HKUDS 官方桌面客户端；
- 安装器发布渠道；
- 自动更新源；
- 后续功能开发位置。

## 当前 Windows 打包边界

正在上游审查的 Windows 打包 PR 只包含必要的基础运行环境：

- 当前 React/FastAPI 应用；
- Electron 桌面宿主；
- 使用哈希锁定依赖的独立 CPython 3.12 运行环境；
- NSIS 安装包构建；
- 与 Windows 当前用户绑定的加密凭据存储；
- 在全新 Windows runner 上执行源码态和打包态生命周期验证。

默认构建明确不包含全部可选 IM/通道依赖、个人微信二维码配对、交易商额外
依赖、自动更新器和 Release feed。

## 仍未完成的发布门槛

1. 完成 Windows 打包 PR 的上游审查与合并。
2. 确定 Authenticode 签名证书由谁持有，以及长期 Release 由谁维护。
3. 在干净 Windows 中验证真正签名后的安装包。
4. 在启用自动更新前，完成一次真实、已签名的 0.3.0 到 0.3.1 升级测试。
5. 继续单独优化体积、冷启动，并评估长期桌面技术路线。

这个归档仓库不发布 EXE。审查阶段生成的未签名安装器只允许留在本机或 CI
中，不能作为面向用户的 Release。

## 许可证与命名

在 HKUDS 明确改变发布归属或命名之前，项目仍使用
**Vibe-Trading Desktop (Unofficial Community Build)**。Vibe-Trading 及其
原始成果归上游作者所有；保留代码继续遵循上游 MIT License，第三方组件
继续遵循各自许可证。
