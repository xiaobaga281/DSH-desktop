# DeepSeek Harness — Windows 桌面版

这是 DeepSeek Harness（基于插件的 agent 内核）的个人 Windows 构建。你只需要装一个 `.exe`，就得到一个原生窗口，在里面跑 agent 会话、多代理团队、电脑操作、内置浏览器和持久记忆，不用开终端。本仓库发布这个桌面构建和它的安装说明，它不是 DeepSeek 的官方发行版。

## 目录

- [下载](#下载)
- [校验文件](#校验文件)
- [安装](#安装)
- [首次启动](#首次启动)
- [数据放在哪里](#数据放在哪里)
- [卸载](#卸载)
- [相对基座改了什么](#相对基座改了什么)
- [已知限制](#已知限制)
- [自己构建](#自己构建)
- [许可与归属](#许可与归属)

## 下载

从本仓库最新的 [release](https://github.com/xiaobaga281/DSH-desktop/releases) 取 `DeepSeek-Harness-Setup.exe`。

| 项目 | 值 |
| --- | --- |
| 文件 | `DeepSeek-Harness-Setup.exe` |
| 大小 | 554,728,771 字节 |
| 应用版本 | `0.1.6-alpha.1` |
| Windows 文件版本 | `0.1.6.1` |
| 适用平台 | x64，Windows 10 及以上 |

## 校验文件

运行之前先比对摘要。下面两条命令都已在这个构建上跑过，返回同一个值。

```powershell
Get-FileHash -Algorithm SHA256 .\DeepSeek-Harness-Setup.exe
certutil -hashfile .\DeepSeek-Harness-Setup.exe SHA256
```

期望的 SHA-256：

```text
13EFF8FE3FADF0F31EA4390E1BF0656CF538C686B71ED56E4EF7974C7C418824
```

对不上就说明你拿到的不是这个构建，不要装。

## 安装

双击安装包按向导走，或者用无交互方式运行。安装是按用户安装的，不需要管理员权限，默认装进你的用户目录。

```powershell
.\DeepSeek-Harness-Setup.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART
```

要换位置就加 `/DIR`。两种写法都在这个构建上实测过两遍：退出码 0、不弹提权框、也不会顺手启动应用。

```powershell
.\DeepSeek-Harness-Setup.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /DIR="D:\DeepSeek-Harness"
```

静默安装会对你机器做的改动：

- 把应用写进安装目录，并在 `HKCU\Software\Microsoft\Windows\CurrentVersion\Uninstall` 下登记一项。
- 在 `Programs\DeepSeek Harness\` 里创建开始菜单项。
- 不创建桌面快捷方式，因为该任务默认不勾选。
- 不启动应用。

## 首次启动

新装的实例没有模型提供方，所以发第一条消息之前先配置：打开设置，进入模型，填一个带 API 地址、协议和密钥的提供方。

在你选定工作区之前，输入框保持不可编辑并显示 `选择一个工作区开始`。这是全新安装的正常状态，不是构建坏了。

## 数据放在哪里

桌面版把所有内容放在 `%LOCALAPPDATA%\DeepSeek Harness`，它绝不读取源码工作树里的数据目录。

| 路径 | 内容 |
| --- | --- |
| `dsh-home\settings.yaml` | 提供方与应用设置 |
| `dsh-home\profiles` | Agent profile 及其安装的包 |
| `dsh-home\skills`、`dsh-home\storages` | 技能与存储实体 |
| `electron-user-data` | Chromium 配置、缓存与单实例锁 |
| `appearance`、`desktop-pet`、`notifications` | 背景、桌宠与通知偏好 |

你的 API 密钥只存在 `dsh-home` 里。要迁移一个配好的实例就拷贝这个目录，并把它当成机密对待。

## 卸载

```powershell
& "$env:LOCALAPPDATA\Programs\DeepSeek Harness\unins000.exe" /VERYSILENT /SUPPRESSMSGBOXES /NORESTART
```

卸载器删除应用目录和它自己创建的开始菜单项。它不删你自己建的快捷方式，也不删 `dsh-home`，所以设置和密钥在卸载后仍然保留。

## 相对基座改了什么

基座项目发布的是 CLI 和插件内核，这个构建把它变成桌面产品。2026-09-24 与同版本纯净基座副本逐文件比对：11,239 个基座文件变成 11,856 个，其中 440 个是改过的基座文件、617 个是这里新增的，没有删除任何文件，全部工作分成 40 个能力面。

| 方面 | 基座没有、这里有的能力 |
| --- | --- |
| 桌面外壳 | 自带窗口装饰的原生窗口、按用户安装、可搬运的包布局、会在通知里点名"完成的是哪个任务"的 Windows 通知身份，以及由渲染侧播放的提示音 |
| 多代理团队 | 内置团队模板、办公室视图、按阶段的计划预算门、每个阶段有 owner |
| 兄弟会话协同 | 通过 `peer_*` 工具互发帖子并配一个 council 面板 |
| 电脑与浏览器操作 | 常驻桌面驱动负责指针与屏幕访问、代理控制指示器、带 provider 深度能力的内置浏览器面板 |
| 记忆与决策 | 蒸馏、检索、固化三态，外加 156 条记录设计理由的仓库内 Agent 笔记 |
| 中文产品界面 | 本地化且重排分区的设置壳、可用模型获取、主题与背景、桌宠，以及带永久删除语义的归档页。市场能读到内置 12 条与 GitHub 20 条技能、MCP 注册表 165 条，安装落在 `dsh-home\skills` 与 `cordis.patch.yml` 的受管区块 |
| 媒体 | 媒体工具，以及按形态而非固定片段时长规划的长视频团队 |
| 升级 | 自包含胶囊，让新基座版本通过真三方合并重放全部定制，配三道登记账和一个升级驱动 |

## 已知限制

- 15 项桌面冒烟跑在这个安装副本上、选好工作区之后的结果是 14 通过、0 失败、1 跳过（见[首次启动](#首次启动)）。剩下那条跳过是工具行检查，需要一个已经调用过工具的会话。
- 这个构建只支持 x64 Windows。
- 它是跟着基座一起维护的个人分支，所以没有升级承诺、没有支持渠道、也没有安全公告流程。

## 自己构建

桌面应用由本分支的源码工作树产出，而不是由本仓库产出。在项目根目录执行：

```powershell
node scripts\ensure-workspace-links.mjs
powershell -File scripts\package-desktop.ps1
```

打包脚本会跑完整构建、离线解析依赖闭包、暂存应用、嵌入图标与版本元数据，并用 Inno Setup 编译安装包，产出 `dist-exe\DeepSeek-Harness-Setup.exe` 和旁边的 portable ZIP。

要把本分支搬到更新的基座版本，跑升级驱动，不要手工合并：

```powershell
.\scripts\upgrade-deepseek-harness.cmd -DryRun -UpstreamRoot "<与当前树同版本的纯净官方源码>"
```

只要胶囊与人读台账不一致，它就拒绝复制任何文件。

## 许可与归属

本仓库为自己发布的内容（安装说明与 release 附件）携带一个 Apache-2.0 的 `LICENSE` 文件。底层 DeepSeek Harness 项目及其源码仍由它自己的许可和维护者管辖。这不是 DeepSeek 的官方产品、官方发行，也不是官方背书。

## Dev Note

本页数字来自 2026-09-24 在一台 x64 Windows 机器上的实测：安装包摘要、静默安装与卸载的实际运行、与纯净基座树的文件数比对，以及跑在安装副本上的桌面冒烟。市场的条目数是从安装副本里读回来的，并且通过它自己的界面真装了一个技能和一个 MCP 服务。双击向导路径和桌面快捷方式任务没有被实际验证。
