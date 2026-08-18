# Changelog

本文件记录 TunnelHub 相对上游基线（edgetunnel 2.1 / 2026-08-11）的改动。上游自身的更新历史见其仓库 CHANGELOG。

## [1.0.0] - 2026-08-18

首个版本。基于 edgetunnel 2.1（2026-08-11 14:45:22）深度改造，项目更名 TunnelHub，聚焦多用户与流量管控。

### ADD — 多用户子账号体系

- 新增 KV `users.json`：子账号字段（name / uuid / enabled / createdAt / expiry / remark / quotaGB），上限 100 个，UUID 必须为 UUIDv4 且不得与主账号或彼此重复
- 认证层多凭证支持：
  - VLESS（`解析魏烈思请求`）多 UUID 字节匹配，返回命中用户
  - Trojan（`解析木马请求`）多用户 sha224 哈希匹配（带缓存）
  - Shadowsocks 入站解密扩展为「用户 × 加密方式」双重探测，出站密钥跟随命中用户
  - WS / gRPC / XHTTP 三个入口统一传递候选 UUID 集，命中后 `yourUUID` 切换，SS 回包加密、XHTTP 回包 padding、链式代理 `/video/` 解密均按命中用户进行
  - WS 早期数据（sec-websocket-protocol）校验支持多凭证
- 每用户独立订阅：`/sub?token=MD5MD5(host+用户UUID)`，命中后以该用户 UUID 生成节点与 TOKEN
- 子账号支持启用/禁用与到期时间，认证与订阅同步失效（缓存约 1 分钟）

### ADD — 月流量配额与用量统计

- `quotaGB` 月度额度（0 / 留空 = 不限，支持两位小数，上限 10240）
- TCP 上下行字节记账：上行写入队列钩子、`forwardataTCP` 首包、`connectStreams` 下行读循环、XHTTP 上行循环与回包计数管道
- 内存聚合 + 刷盘（10 秒定时 / 64MB 定量 / 连接关闭 / 新请求触发）至 KV `traffic.json`，刷盘失败自动回滚内存增量
- 周期为 UTC+8 自然月（`YYYY-MM`），跨月自动清零并解禁
- 超额强制：新连接认证后异步校验拒绝；存量连接块级同步比对，越限立即切断该用户全部活跃连接（活跃连接注册表，WS / gRPC / XHTTP 均接入注销清理）
- 订阅响应 `Subscription-Userinfo`：已用上传 / 下载 / 总额度 / 月底过期时间戳，客户端原生显示

### ADD — 内置子账号管理面板

- `/admin/users` 内置自包含 HTML 管理页（复用上游后台 Cookie 鉴权）：增删改、启停、额度编辑、订阅链接复制、用量展示、单用户/全量流量重置
- `GET /admin/users.json`：列表 + subToken + upBytes / downBytes / usedBytes / quotaBytes / overQuota
- `POST /admin/users.json`：保存（服务端规范化校验：UUID 格式 / 去重 / 到期格式 / 额度范围）
- `POST /admin/traffic.json`：`{"action":"reset","uuid":...}` 或 `{"action":"resetAll"}`

### Fix（开发过程中发现并修复）

- 流量刷盘快照改用深拷贝：`new Map(内存表)` 浅拷贝导致扣减内存时连带清零快照，KV 写入空记录

### Docs

- 重写 README（差异化定位 / 部署文档 / 环境变量 / 子账号与配额使用说明）
- 新增 SAFETY.md（法律风险声明 + 作者安全指南）、NOTICE.md（GPL-2.0 衍生作品声明与改动清单）
