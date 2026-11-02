# x230-ops — 一台 12 年老笔记本上的自托管运维工具箱

> 硬件：ThinkPad X230（i5-3230M / 7.6G RAM / 240G SSD / zram）
> 角色：7×24 常驻，跑自建 Agent 服务、AI CLI 网关、邮件与站点健康巡检。

这个仓库是我日常在用的几个小工具的合集，都是**几百行、无第三方依赖、可直接抄**的脚本。

## 工具一览

| 工具 | 一句话 | 依赖 |
|---|---|---|
| `bin/infra-check` | 12 项并行巡检：系统 / 自建服务 / 账号链路 / 网络 / 远端 SSH，8 秒出结果 | Python 3 + curl + ssh |
| `bin/x230-tune` | 服务器针对性优化：缓存、coredump、journal 上限、swappiness、THP、IO 调度、内存护栏 | Python 3 + sudo |
| `bin/agy-bridge` | 把 Antigravity CLI（`agy`）包装成 OpenAI 兼容接口，供自建 Agent 调用 | Python 3 |
| `bin/agy-usage` | 解析 `agy /usage`，输出双额度池（Gemini / Claude）剩余百分比 | Python 3 |
| `bin/wp-post` | 把 Markdown 一键发到 WordPress（REST API + 应用密码），支持草稿/发布 | Python 3 + markdown |

## infra-check：8 秒做一次全栈体检

```text
== infra-check 2026-11-02 21:13 ==
 ✓ 系统 负载0.21 内存3.0/7.6G 磁盘16% 温度52°C
 ✓ dsh-web :3080        http=401（活着，要认证）
 ✓ agy-bridge :8899     13 个模型
 ✓ WordPress 站点        http=200
 ✓ WP REST API          http=200
 ✓ Gmail IMAP           16 封
 ✓ Antigravity 额度     Gemini 周99 / Claude 周96
 ✓ Clash 代理 :7897     可连
 ✓ 出口 IP              133.18.114.145
 ✓ 香港服务器 SSH        OK
 ✓ Google               http=200
 ✓ OpenAI chatgpt       http=200
== 全部正常 ==
```

- 12 项**并发**执行，总耗时 ~8.5 秒；
- 退出码分级：`0` 全好 / `1` 有告警 / `2` 关键项故障，可直接挂 cron；
- `--json` 给别的脚本消费，`--quiet` 只打失败项，`--push` 异常时推 ntfy。

## x230-tune：老机器上的针对性优化

```bash
x230-tune              # 审计
x230-tune --apply      # 一键优化（幂等）
x230-tune --aggressive # 额外清空 pacman 缓存
```

实测在这台机器上做了这些（都有持久化配置）：

| 动作 | 效果 |
|---|---|
| 停用空跑的 docker/containerd/socket | 服务 27 → 25，省 ~95MB |
| 清 coredump | 41MB → 1MB |
| journald 上限 200MB | 日志不再无限增长 |
| 清 Chromium profile 缓存 | 释放 147MB（登录态保留） |
| `vm.swappiness` 180 → 100 | zram 场景更稳 |
| THP `always` → `madvise` | 减少内存合并带来的抖动 |
| SATA SSD 调度器 `mq-deadline` → `none` | 少一层 IO 排队 |
| agy-burn ≤1.5G / agy-bridge ≤600M | 后台任务挤不动主服务 |

配套 `x230-tune.timer`（每周日 04:30）自动维护，`browse` 到的部分全部可回滚。

## agy-bridge：把订阅制 AI CLI 变成 API

```text
DSH / 自建 Agent ──(OpenAI 协议)──> agy-bridge :8899 ──> agy -p --model <模型>
```

- `GET /v1/models` 列出 agy 可用模型（Gemini / Claude 系列）；
- `POST /v1/chat/completions` 把 messages 压平成一段 prompt 交给 `agy -p`；
- 支持 `stream`（SSE，整段返回）；
- **边界要说清**：`agy -p` 是一次性 agent 调用，没有多轮状态、也不支持 tool-calling，
  适合写作、翻译、第二意见，不适合当需要读写文件的编码主模型。

## 现在跑在什么上

- Arch Linux / systemd / 25 个服务；
- 自建 DSH（Agent 运行框架）+ 飞书通知；
- 香港 VPS 做出口与镜像测速对照；
- 全部用 systemd 管理，定时任务失败会推 ntfy。

## 许可

MIT。脚本里的路径/凭据都从环境或 `~/.config/secrets/` 读取，仓库里**不含任何密钥**。
