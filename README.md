# 🚇 TunnelHub

> 基于 Cloudflare Workers 的**多用户**边缘隧道管理与配额系统 —— 子账号分配、月流量额度、用量统计，一个文件全部搞定。

[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](./LICENSE)
[![Upstream](https://img.shields.io/badge/Based_on-edgetunnel-orange.svg)](https://github.com/cmliu/edgetunnel)

---

## ⚠️ 风险警告（务必阅读）

> [!IMPORTANT]
> **本项目仅供学习、研究与教育目的。**
>
> 1. **中国大陆法律风险**：在中国大陆，未经国家批准擅自使用非法信道进行国际联网（俗称"翻墙"）属于**违法违规行为**，可能面临警告、罚款等行政处罚，情节严重的可能承担更严重的法律后果。本项目**不以任何方式鼓励、教唆或协助**此类行为。
> 2. **使用者责任自负**：本项目按"现状"提供，不提供任何明示或默示的担保。部署、使用本项目产生的一切后果由使用者自行承担，与作者无关。
> 3. **平台条款风险**：通过 Cloudflare Workers 中转大量非网页流量可能违反 [Cloudflare 服务条款](https://www.cloudflare.com/zh-cn/terms/)，存在账号被封禁、服务被终止的风险。
> 4. **滥用风险**：请勿将本项目用于任何违反所在地区法律法规的用途；请勿把子账号分配给不信任的人——**他们的流量走你的 Cloudflare 账号**。
> 5. 建议仅用于学习边缘计算、网络协议与 Workers 开发技术，测试完成后及时删除部署。

完整声明见 [SAFETY.md](./SAFETY.md)。

---

## 📖 项目简介

**TunnelHub** 与社区中常见的单账号隧道脚本不同，它面向"**一个管理员 + 多个使用者**"的小团队 / 家庭 / 朋友间共享场景，把"用户管理"和"流量管控"作为一等公民：

| 能力 | 说明 |
| :--- | :--- |
| 👥 **子账号体系** | 管理员可为每个使用者创建独立子账号（独立 UUID），各自拥有独立订阅链接，与主账号完全隔离 |
| 📊 **月流量额度** | 为每个子账号设置月度流量上限（如 50GB / 100GB / 200GB），按北京时间自然月自动重置 |
| ⛔ **超额强制** | 额度用尽后：新连接直接拒绝、存量连接立即切断，不留死角 |
| 📈 **用量统计** | TCP 上下行字节级统计，管理页实时查看每人已用流量；订阅响应携带标准 `Subscription-Userinfo`，客户端原生显示用量与到期 |
| 🗓️ **到期管理** | 支持为子账号设置有效期，到期自动失效 |
| 🖥️ **内置管理面板** | `/admin/users` 一个页面完成增删改、启停、复制订阅、重置流量 |
| 🛡️ **全协议继承** | 完整继承上游的 VLESS / Trojan / Shadowsocks 协议与 WS / gRPC / XHTTP 传输、优选订阅、链式代理等能力 |

### ✨ 核心特性（继承自上游）

- **协议支持**：VLESS、Trojan、Shadowsocks，多传输方式（WebSocket / gRPC / XHTTP）
- **订阅系统**：自动订阅生成与混淆转换，适配 Clash / Sing-box / Surge / v2rayN 等主流客户端
- **性能加速**：自定义 ProxyIP、SOCKS5/HTTP 链式代理、优选 API、TCP 并发拨号
- **部署灵活**：完整适配 CF Workers 与 CF Pages（上传 / GitHub）

---

## 🚀 快速部署

> 前置要求：一个 Cloudflare 账号。**KV 绑定是子账号与流量功能的必要条件。**

### Workers 部署

1. 下载本仓库的 [`_worker.js`](_worker.js)，在 Cloudflare 控制台创建一个新 Worker，将文件内容粘贴进编辑器；
2. `设置` → `变量` → 添加变量 **`ADMIN`**，值为你的管理员密码（请使用强密码，它同时派生主账号凭证）；
3. `绑定` → 添加 **KV 命名空间**绑定，变量名必须填 **`KV`**；
4. `触发器` → 添加自定义域（例如 `mt.your-domain.com`），等待证书生效；
5. 访问 `https://mt.your-domain.com/admin` 输入管理员密码登录。

### Pages 部署（上传 / GitHub 两种方式均可）

与 Workers 类似：上传本仓库 zip 或连接 GitHub 仓库 → 设置环境变量 `ADMIN` → 绑定 KV（变量名 `KV`）→ 绑定自定义域。

### 创建你的第一个子账号

1. 登录后台后访问 **`/admin/users`**；
2. 「＋ 新增子账号」→ 填名称（UUID 可留空自动生成）→ 可选填月流量额度（GB）、到期时间 → 「保存全部修改」；
3. 点击该子账号的「复制订阅链接」发给使用者，对方导入 v2rayN / Clash 等客户端即可，客户端内可直接看到用量与额度。

---

## 🔑 环境变量说明

| 变量名 | 必填 | 示例 | 备注 |
| :--- | :---: | :--- | :--- |
| **ADMIN** | ✅ | `123456` | 管理面板密码，同时派生主账号 UUID |
| **KEY** | ❌ | `CMLiussss` | 快速订阅路径密钥 / 加密密钥 |
| **UUID** | ❌ | UUIDv4 | 强制固定主账号 UUID（仅主账号） |
| **PROXYIP** | ❌ | `proxyip.xxx:443` | 全局自定义反代 IP |
| **HOST** | ❌ | `mt.example.com` | 指定域名（多域名时影响订阅 TOKEN 计算） |
| **URL** | ❌ | `https://...` | 主页伪装地址 |
| **GO2SOCKS5** | ❌ | `*xxx.com` | 强制走 SOCKS5 的名单 |
| **DEBUG** | ❌ | `1` | 开启调试日志 |
| **OFF_LOG** | ❌ | `1` | 关闭 KV 日志记录 |
| **BEST_SUB** | ❌ | `1` | 开启优选订阅生成器 |
| **PRELOAD_RACE_DIAL** | ❌ | `1` | 预加载竞速拨号 |
| **TCP_CONCURRENT_DIAL** | ❌ | `2` | TCP 并发拨号数 |
| **PROXY_CONCURRENT_DIAL** | ❌ | `1` | 反代并发拨号数 |

---

## 👥 子账号管理

- **入口**：后台 `/admin/users`（内置页面，无需外部依赖）
- **认证方式**：子账号以自身 UUID 作为 VLESS 的 UUID、Trojan 的密码、Shadowsocks 的密码
- **订阅链接**：`https://域名/sub?token=<该用户专属token>`，与主账号订阅格式完全一致，客户端无感知
- **数量上限**：默认 100 个（存储于 KV `users.json`）
- **生效延迟**：禁用 / 到期 / 删除 / 调整额度后，服务端缓存最长约 1 分钟内生效

### 月流量额度

- 「流量额度(GB)」列设置月度上限，留空或 `0` 表示不限；
- 统计 TCP 上下行原始字节，按**北京时间自然月**自动清零（存储于 KV `traffic.json`）；
- 超额后新连接被拒、存量连接数秒内切断；管理页可按用户「重置流量」或「清空全部流量」立即解禁；
- 为**近似统计**（内存聚合 + 定期刷盘，多实例下存在少量误差），适用于额度管控，**不适用于计费**；

### 管理接口（需后台登录 Cookie）

| 接口 | 说明 |
| :--- | :--- |
| `GET /admin/users.json` | 子账号列表（含订阅 token、已用流量、额度） |
| `POST /admin/users.json` | 保存子账号列表 |
| `POST /admin/traffic.json` | 重置流量：`{"action":"reset","uuid":"..."}` 或 `{"action":"resetAll"}` |

---

## 💻 客户端适配

| 平台 | 推荐客户端 |
| :--- | :--- |
| **Windows** | v2rayN、Clash Verge Rev、mihomo-party、FlClash |
| **Android** | v2rayNG、ClashMetaForAndroid、FlClash |
| **iOS** | Shadowrocket、Surge、Stash、Loon |
| **macOS** | Clash Verge Rev、Surge、FlClash |

---

## 🙏 致谢与开源许可

本项目是基于以下开源项目的**衍生作品**（derivative work），深受其代码启发并在其基础上改造而来：

### 🌟 直接上游

- **[cmliu/edgetunnel](https://github.com/cmliu/edgetunnel)**（GPL-2.0）—— 本项目以其 2.1 版本为基础进行深度改造，协议实现、传输层、订阅系统与管理面板架构均来源于它。**没有这个项目就没有 TunnelHub。**

### 🛠 上游的致谢名单（同样适用于本项目）

- [zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)、[3Kmfi6HP/EDtunnel](https://github.com/6Kmfi6HP/EDtunnel)、[SHIJS1999/cloudflare-worker-vless-ip](https://github.com/SHIJS1999/cloudflare-worker-vless-ip)、[Stanley-baby](https://github.com/Stanley-baby)
- [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR/tree/master/Clash/config)、[Mingyu](https://github.com/ymyuuu/workers-vless)、[ToiCF](https://github.com/ToiCF)、[eooce](https://github.com/eooce/Cloudflare-proxy)
- [Sukka](https://ip.skk.moe/)、[xream](https://github.com/xream)、[zhangtaile](https://github.com/zhangtaile)、[1345695](https://github.com/1345695) 等
- 以及 edgetunnel 社区的所有贡献者

### 📄 许可证

本项目继承并遵守 **[GPL-2.0](./LICENSE)** 许可证：

- 本项目作为衍生作品，整体以 GPL-2.0 授权发布；
- 上游及其贡献者保留其对原始代码的全部版权；
- TunnelHub 的改动部分（多用户子账号体系、月流量配额、用量统计等）同样以 GPL-2.0 授权，任何人自由使用与再分发时须遵守 GPL-2.0 并保留署名；
- 衍生关系与改动清单详见 [NOTICE.md](./NOTICE.md)。

---

## ⚠️ 免责声明

1. 本项目（"TunnelHub"）仅供**教育、科学研究及个人安全测试**之目的；
2. 使用者在下载或使用本项目代码时，必须严格遵守所在地区的法律法规；在中国大陆，擅自进行国际联网属违法违规行为，请勿将本项目用于此类用途；
3. 作者对任何滥用本项目代码导致的行为或后果均不承担任何责任；
4. 本项目不对因使用代码引起的任何直接或间接损害负责；
5. 建议在测试完成后 24 小时内删除本项目相关部署。

---

**如果你觉得本项目（或上游 edgetunnel）对你有帮助，请给两边都点一个 Star 🌟 —— 特别请优先支持上游作者。**
