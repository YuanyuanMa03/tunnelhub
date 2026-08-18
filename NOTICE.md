# NOTICE — 衍生作品声明

本项目 **TunnelHub** 是 [cmliu/edgetunnel](https://github.com/cmliu/edgetunnel)（GPL-2.0）的衍生作品（derivative work）。

## 版权归属

- 原始代码的版权由 **edgetunnel 的作者及全体贡献者** 保留；
- TunnelHub 维护者对其新增 / 修改部分的代码同样以 GPL-2.0 授权（见 [LICENSE](./LICENSE)），不主张对上游代码的任何权利；
- 本项目整体作为 GPL-2.0 授权作品分发，任何人再分发或修改须遵守 GPL-2.0 的全部条款，并保留本声明。

## 基于的版本

- 上游基线：edgetunnel `2.1`（`2026-08-11 14:45:22`，见上游 CHANGELOG）

## TunnelHub 的主要改动（相对上游基线）

以下改动由 TunnelHub 维护者完成，同样以 GPL-2.0 授权：

### 1. 多用户子账号体系

- 新增 KV `users.json` 子账号存储（名称 / UUID / 启用状态 / 到期时间 / 备注，上限 100 个）；
- VLESS / Trojan / Shadowsocks 认证层改造为多凭证匹配（`解析魏烈思请求` / `解析木马请求` / SS 入站密钥探测支持多用户），WS / gRPC / XHTTP 三个入口统一生效；
- 每个子账号独立订阅链接（`/sub?token=MD5MD5(host+uuid)`），按命中用户生成节点；
- XHTTP 回包 padding 标识、链式代理 `/video/` 路径解密按命中用户切换。

### 2. 月流量配额与用量统计

- 子账号可设置 `quotaGB` 月度流量额度（留空 / 0 = 不限）；
- TCP 上下行字节级记账（上行写入队列、`forwardataTCP` 首包、`connectStreams` 下行、XHTTP 回包管道）；
- 内存聚合 + 定时 / 定量刷盘至 KV `traffic.json`，按 UTC+8 自然月自动重置；
- 超额强制：新连接认证后拒绝，存量连接块级比对越限即切断（活跃连接注册表）；
- 订阅响应携带 `Subscription-Userinfo`（用量 / 额度 / 月底过期）。

### 3. 内置子账号管理面板

- 新增 `/admin/users` 内置管理页与 `GET/POST /admin/users.json`、`POST /admin/traffic.json` 接口（复用上游后台 Cookie 鉴权）。

### 4. 其他

- 版本标识与文档（README / SAFETY / CHANGELOG）重写，项目更名为 TunnelHub。

## 继承但未修改的部分

协议实现（VLESS / Trojan / Shadowsocks / SOCKS5 / HTTP / TURN / SSTP）、传输层（WS / gRPC / XHTTP / Grain 合包）、优选订阅、订阅转换、TG / CF 用量集成、登录与后台框架等均来自上游代码，未经实质修改。

## 上游的致谢

edgetunnel 自身的致谢名单（zizifn/edgetunnel、EDtunnel、ACL4SSR、Sukka、xream 等）在 TunnelHub 中继续有效，完整名单见 [README](./README.md#-致谢与开源许可)。
