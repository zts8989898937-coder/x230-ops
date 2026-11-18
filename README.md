# x230-ops — 一台 12 年老笔记本上的自托管运维工具箱

> 硬件：ThinkPad X230（i5-3230M / 7.6G RAM / 240G SSD / zram）
> 角色：7×24 常驻，跑自建 Agent 服务、AI CLI 网关、邮件与站点健康巡检。

这个仓库收录了我日常使用的运维工具。每个脚本可以独立阅读，依赖和部署条件见下表。

## 工具一览

| 工具 | 一句话 | 依赖 |
|---|---|---|
| `bin/infra-check` | 12 项并行检查，覆盖系统、应用服务、网络和远端 SSH | Python 3 + curl + ssh |
| `bin/x230-tune` | 服务器针对性优化：缓存、coredump、journal 上限、swappiness、THP、IO 调度、内存护栏 | Python 3；应用修改需相应系统权限 |
| `bin/agy-bridge` | 把 Antigravity CLI（`agy`）包装成 OpenAI 兼容接口，供自建 Agent 调用 | Python 3、已配置的 `agy` |
| `bin/agy-usage` | 解析 `agy /usage`，输出双额度池（Gemini / Claude）剩余百分比 | Python 3、已配置的 `agy` |
| `bin/wp-post` | 把 Markdown 一键发到 WordPress（REST API + 应用密码），支持草稿/发布 | Python 3 + markdown |

## 使用说明

- **巡检**｜按自己的环境配置 `bin/infra-check` 中的检查目标，再运行脚本。`--json` 输出结构化结果，`--quiet` 只显示异常项，`--push` 使用环境中的通知配置。退出码 `0` 表示通过，`1` 表示有告警，`2` 表示 SSH 或代理检查失败。
- **系统维护**｜`bin/x230-tune` 默认进行审计。`--apply` 会执行系统改动，`--aggressive` 会额外清理 pacman 缓存。应用前请检查脚本中的机器和服务假设。维护范围包括日志空间、崩溃转储、缓存、内核参数与资源上限。
- **CLI 网关**｜`bin/agy-bridge` 提供 `GET /v1/models` 和 `POST /v1/chat/completions`。默认监听回环地址。每次请求调用一次 `agy -p`，可使用 SSE 返回整段回答。对话上下文由调用端随请求提供。
- **WordPress 发布**｜`bin/wp-post` 读取 Markdown 文件和本机的应用密码配置。默认建立草稿，显式指定 `--status publish` 时才公开发布。

## 许可

[MIT](LICENSE)

## 2026-11-18 公开版更新

将巡检目标替换为示例值，保留巡检、系统维护、CLI 网关和 WordPress 发布工具。 [查看更新提交](https://github.com/zts8989898937-coder/x230-ops/commit/ca1c9f249d37ccbdbd905238ff581c5bcf976bee)

[公开范围](PUBLIC_SCOPE.md) · [架构说明](docs/ARCHITECTURE.md) · [后续路线](docs/ROADMAP.md)
