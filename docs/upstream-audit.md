# 上游源码审计

审计日期：2026-09-22。方法：公开仓库指定提交的静态阅读。本次未运行客户端、未访问员工账号、未验证真实供应商响应。以下不是兼容性认证。

## 冻结基线

| 项目 | 阅读提交 | 查询到的最新发布 |
|---|---|---|
| steipete/CodexBar | `b99a91694d7878202ee0b6d0bc725952bbd8c328` | v0.64.0，2026-09-21 |
| nesszer/Win-CodexBar | `8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056` | v0.60.3，2026-09-15 |

提交基线与发布包不等价。Windows 阅读提交比发布日期新，不能承诺发布包已有主分支的新能力。Windows README 的旧版本描述也不能代替 Releases 和实机版本。

## 证据与结论

| 观察 | 源码证据 | 对本项目的影响 |
|---|---|---|
| Mac 有 usage/cost JSON、serve 等 CLI 入口 | [Mac CLI](https://github.com/steipete/CodexBar/blob/b99a91694d7878202ee0b6d0bc725952bbd8c328/docs/cli.md) | 可做独立适配器，不必先 fork UI |
| Windows 有独立 CLI，打包名称不同于 GUI | [Windows CLI](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/docs/CLI.md) | 发现和版本协商按平台实现 |
| usage hook 主要是额度状态，存在节流 | [Mac hook](https://github.com/steipete/CodexBar/blob/b99a91694d7878202ee0b6d0bc725952bbd8c328/Sources/CodexBarCore/Hooks/HookEvent%2BUsageUpdated.swift)、[Windows transition](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/rust/src/core/hook_transition.rs) | 不是每请求 token 事件；约600秒节流不能当使用时长 |
| Mac 成本模型有日级、缓存、模型分解和覆盖信息 | [CostUsageModels](https://github.com/steipete/CodexBar/blob/b99a91694d7878202ee0b6d0bc725952bbd8c328/Sources/CodexBarCore/CostUsageModels.swift) | 必须保留原语义、覆盖及费用来源 |
| Windows cost 契约是独立结构 | [cost CLI](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/rust/src/cli/cost.rs)、[summary contract](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/rust/src/codex_costs/summary_contract.rs) | `tokens.cached` 等不能不核语义直接套 Mac 字段 |
| 历史 Claude 成本可能无法归属账号 | [cost scanner](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/rust/src/cost_scanner.rs) | 当前登录账号不能认领旧日志 |
| Windows dashboard snapshot 部分字段恒为 null | [dashboard snapshot](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/rust/src/cli/serve/dashboard/snapshot.rs) | credits/status 空值不等于余额或健康为零；必要时选择其他已验证入口 |
| session 有开始/最后活动和活跃判断 | [Mac AgentSession](https://github.com/steipete/CodexBar/blob/b99a91694d7878202ee0b6d0bc725952bbd8c328/Sources/CodexBarCore/AgentSession.swift)、[Windows sessions](https://github.com/nesszer/Win-CodexBar/blob/8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056/rust/src/agent_sessions.rs) | 活跃启发式不是工作时长；不直接上传会话内容 |

serve 是本地访问入口，不是员工管理后端。即使支持非回环绑定和认证，也不应把员工原工具直接暴露公网。本项目仅由 Collector 主动出站到公司服务。

未发现可直接复用的完整员工邀请、组织权限、购买账本、员工授权、跨设备去重与企业报表体系。新增这些是本项目职责，不应宣传为两个上游已有功能。

## 许可与边界

两仓库冻结基线均为 MIT。Mac 许可证署名为2026 Peter Steinberger；Windows 许可证署名为2025 Peter Steinberger，其 NOTICE 还说明部分 codexcontrol 衍生代码。见 [第三方说明](../THIRD_PARTY_NOTICES.md)。许可允许的代码使用不等于供应商接口、商标或员工数据处理均已获授权。

## 进入实现前必须补证

对拟支持的发布包记录版本、文件摘要、OS、安装渠道；在授权测试账号上逐个抓取脱敏 fixture；核对 JSON、单位、时区、缓存语义、账号切换和退出登录行为。禁止因为主分支包含字段，就把全部旧版本标为支持。
