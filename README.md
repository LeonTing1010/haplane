# HapLane · 鸿蒙 HAP 一键签名 / 调试 / 部署

> **一条命令，穿过签名墙。** CLI + MCP，agent 可驱动，装完自动验签。
>
> `适配 HarmonyOS NEXT · OpenHarmony` ｜ `手机 / 平板 / 穿戴 / 智慧屏` ｜ [`v0.3.0 已发布 ⬇`](https://gitee.com/LeonTing1010/haplane/releases/v0.3.0) ｜ `无需破解 · 走你自己的账号签名`

🌐 **官网**：https://leonting1010.github.io/haplane/

---

纯血鸿蒙装个 HAP，能卡你一下午——华为账号、UDID、配额、证书链、system 权限报错，**一步错就装不上**。

HapLane 把这七步折成一条命令：

```console
$ haplane deploy ./app.hap
› 连接设备        hdc  ✓  UDID 454D5504…
› 读取包名        com.example.app  ✓
› 生成签名材料    OpenHarmony 证书链  ✓
› 签名            sign-app  ✓
› 安装            system 权限报错 ⚠   自动降级 → 重装 ✓
› 验签            valid: true ✓
✓ 已装上设备 · 用时 6.2s
```

想接 CI、让 AI agent 跑也行，先图一键就够。

## 接入 MCP：挂进你的 AI，用说的

HapLane 是 **MCP 工具**——不用背命令，agent 自己选工具、撞错自己修。三步接上：

**1. 下载 MCP（Gitee 直连，不走 npm）**：[`haplane-mcp.mjs` ⬇](https://gitee.com/LeonTing1010/haplane/releases/download/v0.3.0/haplane-mcp.mjs)（36 KB，放到本地任意路径）

**2. 粘进你的 MCP 配置**：

```json
{
  "mcpServers": {
    "haplane": {
      "command": "node",
      "args": ["/你的路径/haplane-mcp.mjs"]
    }
  }
}
```

**3. 在 AI 里直接说**：「帮我把 `./app.hap` 签名装到手机」——agent 自己调 `deploy_to_device`，撞 system 权限自动降级重装，装完验签。

- **支持**：Claude Code / Cursor / Cline / DevEco Code / 任意 MCP 客户端
- **11 个工具**（agent 自己选）：`deploy_to_device` `sign` `verify` `install` `doctor` `gen_signing_material` `agc_provision` `agc_status` `rename_bundle` `hdc_devices` `hdc_udid`
- 需 Node ≥ 20（做鸿蒙开发一般都有，hvigor 就依赖它）

## 也能命令行直接用

一条 `deploy` 搞定，也能拆成四步——只想签名、或只想推包，单独调那一步就行。

| 命令 | 做什么 | 要连设备？ |
|---|---|---|
| `haplane deploy --in app.hap` | 签名 → 装机 → 验签，一条龙（**带 `--in` 跳过打包**） | 要（装机那步） |
| `haplane deploy` | 从源码工程走全链：`hvigorw` 打包 → 签名 → 装机 → 验签 | 要 |
| `haplane sign --in app.hap` | 只签名，产出签好的 HAP | 上架 / CI 不用；debug 真机连一次取 UDID |
| `haplane install --in app.hap` | 只推已签好的包（≈更聪明的 `hdc install`） | 要 |
| `haplane verify --in app.hap` | 只验签名，判真假 | 不用 |
| `haplane setup` / `doctor` / `device` | 生成配置 / 体检工具链 / 列设备取 UDID | `device` 要 |

## 签名墙不是难，是"烦"

手动侧载一个 HAP，你得走完这七步：

1. 开发者模式 + USB/无线调试 + 手机弹窗授权
2. `hdc` 连真机，取 UDID
3. AGC 注册设备、申请 debug 证书（配额 2 本/年）
4. 申请 Profile，绑死 bundleName + UDID
5. 包名被占？逐用户改名
6. `hap-sign-tool` 签名（路径不能有中文）
7. 装机 → system 权限报错 → 卡住

HoKit、小白助手是把这些塞进界面里一个个点；**HapLane 是一条命令跑完，还能写进脚本、让 AI 帮你跑**。骨架已在模拟器五步全绿实测（hdc → sign → install → verify）。

## 最容易卡的三件事

| | 卡点 | HapLane 怎么处理 |
|---|---|---|
| **认证** | 寄生会话 / cookie 会话 / 机器密钥（client_credentials）该用哪个？配额墙、包名全局占用（装 B站撞过 `205339064` 吧？） | 文档讲透，配套《签名墙通关手册》 |
| **system 权限** | 装 system 应用几乎都会撞权限报错 | 自动把 apl 降到能装的档再重装，不用你手改配置 |
| **穿戴大包** | watch 表盘、5MB 以上大包推送，别的工具容易卡 | 单独处理这条路 |

丢一个装不上的 HAP，`haplane doctor` 会真诊断：包名被占？覆盖装撞 `9568332`（签名不一致）？system 权限不够？——不是给你一句"AI 分析中"就完了。

## 和别的工具比

| 能力 | HapLane | DevEco Code | HoKit | HapViewer |
|---|:---:|:---:|:---:|:---:|
| 无头 CLI（无 GUI） | ✓ | — | — | — |
| 接任意 MCP agent | ✓ | 仅自家 | — | — |
| CI 流水线集成 | ✓ | — | — | — |
| system 权限自动降级 | ✓ | — | — | — |
| 穿戴大包推送 | ✓ | — | — | — |
| HAP 深度解析 | 基础 | ✓ | 基础 | ✓ |
| 投屏 / 消费级 GUI | — | — | ✓ | — |
| 开源免费 | 开源可查·MIT | — | 闭源免费 | ✓ |

> 对比基于各工具公开能力与实测；空位表示未提供或非其定位，非贬义。HapViewer 看包强、HoKit 有投屏 GUI、DevEco Code 是华为的 IDE 助手，各有各的用处。

## 老实说：它不做什么

**不做**

- 不破解、不绕过签名——只帮你把该走的流程走顺
- 只处理你自己或已授权的设备 / 账号
- 不碰纯血鸿蒙真机的商用根：真机走你自己的 AGC 材料

**前提**

- 华为开发者账号 + AGC（真机 / 上架签名要）
- 华为 SDK：`hdc` / `hap-sign-tool` / `java`（各平台官方版都有）
- Node（或用单文件二进制）；macOS / Windows / Linux 都行

## 上手

**下载**（走 Gitee Release，国内直连，不走 npm）：[**v0.3.0 ⬇**](https://gitee.com/LeonTing1010/haplane/releases/v0.3.0)

| 文件 | 适用 | 要求 |
|---|---|---|
| **`haplane.mjs`**（22 KB，推荐） | **Windows / Linux / macOS 通用** | Node ≥ 20（做鸿蒙开发一般都有，hvigor 就依赖它） |
| `haplane-macos-arm64` | macOS Apple Silicon | 免装 Node，下载即用 |

```console
$ node haplane.mjs setup                   # 首次：探测 SDK + 生成配置与 .env 模板
#   把签名密码写进 .env（HAP_KEY_PWD / HAP_KEYSTORE_PWD，密码只从环境变量读）
$ node haplane.mjs doctor                  # 体检：工具链与签名材料是否就位
$ node haplane.mjs deploy --in ./app.hap   # 对现成 HAP：签名 → 装机 → 验签
```

加 `--dry-run` 只打印将执行的命令、不真跑——没真机也能先看全链。

> ⚠️ **老实说**：本版帮助文本里仍自称 `hap-deploy`（内部名），命令行为与上面一致，下版统一为 `haplane`。Windows / Linux 的免 Node 原生二进制待 CI 补。

**签名踩坑、穿戴推送、AGC 配额这些，现在就能问**，不用等发布——开 [Issue](https://gitee.com/LeonTing1010/haplane/issues) 一起解，或先翻[《签名墙通关手册》](docs/签名墙通关手册.md)。要帮你把 app 签好、上架、代交付，也可以直接找我。

---

**开源可查（MIT）**：工具代码 [Gitee](https://gitee.com/LeonTing1010/hap-deploy) · [GitHub](https://github.com/LeonTing1010/hap-deploy-cli)。向 Fastlane 致敬——工具免费开源，价值在交付与手册。HapLane 只处理你自己或已授权设备 / 账号的 HAP 签名与部署，不绕过任何非授权系统的安全控制。
