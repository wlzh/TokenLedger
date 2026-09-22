# 逻辑数据模型

状态：评审草案；仅描述未来实体和约束，无已创建数据库，无 DDL/DML。

## 1. 身份与授权

| 实体 | 核心字段 | 约束 |
|---|---|---|
| organizations | id, name, timezone, policy_version | 首版单组织部署，仍显式隔离所有业务数据 |
| users | id, login_identity, display_name, status | 登录身份与供应商邮箱分离 |
| memberships | organization_id, user_id, role, status | 邀请接受后激活；离职不复用原成员ID |
| invitations | id, organization_id, token_digest, invited_identity, expires_at, accepted_at | 一次性，目标身份绑定 |
| devices | id, membership_id, public_name, platform, agent_version, state | 名称由员工设置，不默认上传系统用户名/主机名 |
| device_credentials | device_id, secret_digest, expires_at, revoked_at | 专用采集凭据，不存 AI 凭据 |
| consent_versions | id, membership_id, device_id, destination, selected_scopes, policy_version, effective_at, revoked_at | 授权新增范围须本人确认 |
| source_scopes | id, device_id, provider, source_kind, capability_version, declared_coverage | scope UUID 本地生成，不用真实文件路径 |
| provider_accounts | id, membership_id, provider, account_alias, identity_fingerprint?, account_scope | 同人多账号，脱敏邮箱不是唯一键 |
| account_bindings | source_scope_id, account_id?, valid_from, valid_until, attribution, consent_id | 历史不被当前登录状态覆盖 |

账号跨设备合并应由员工确认。允许服务商返回的非密钥账号标识做本地映射；未经授权不上传原标识。指纹不能由 API Key 或 Cookie 派生，凭据轮换不能生成新业务账号。

混合日志根若无法隔离账号，只有员工额外确认整个选定日志范围可上报才允许以 unassigned/employee scope 上报；未确认就禁用成本扫描，而不是先上传再让管理者筛。

## 2. 用量实体

| 实体 | 核心字段 | 约束 |
|---|---|---|
| ingest_batches | device_id, stream_id, batch_id, sequence, content_digest, received_at, state | 同 batch 相同内容返回原结果；不同内容冲突 |
| source_revisions | source_scope_id, period, metric_family, revision, provenance | 修订源于固定范围，不按上传时刻累加 |
| quota_observations | source_scope_id, account_id?, observed_at, window_id, cadence?, reset_at?, ratio?, unit? | 快照不能 sum；原字段未知保留未知 |
| usage_summaries | source_scope_id, period_start/end, bucket_timezone, model_key?, token_fields, coverage, attribution | 同逻辑键 revision 单调；父级total和模型细分不是两套加项 |
| usage_events | source_namespace, pseudonymous_event_id, occurred_at, model_key, token_fields | 后期精细来源；稳定 ID 才跨设备去重 |
| activity_intervals | membership_id, source_scope_id, start/end, method_version | 后期估算；不冒充工时 |
| data_health | source_scope_id, last_attempt/success, source_updated_at, reason_code, gap_intervals | 不存原始异常/路径/响应 |
| report_runs | id, filters, metric_version, price_version, generated_at | 导出附口径、截止时间和修订版本 |

period 使用半开区间 `[start,end)`。计数数据库用受限非负整数，金额用高精度 decimal；无值为 NULL，不设默认0。超大值在入口拒绝。

汇总层由源记录重算或事务维护，不以 HTTP 请求数作为计量因子。删除、修订、换源均应触发相关物化结果重算。

## 3. 购买与费用实体

| 实体 | 内容 | 边界 |
|---|---|---|
| subscriptions | employee, product, provider_account?, plan_declared/observed, billing_mode, service_period, renewal | 手填与观测套餐分开，变化不抹历史 |
| purchases | subscription_id?, amount, currency, payer, purchased_at, service_period, declared_by | 新增购买不自动成为已审核支出 |
| company_allocations | purchase_id, company_share, currency, basis | 公司承担金额与员工个人承担区分 |
| payment_events | purchase_id, payer/payee role, payment_kind, amount, currency, paid_at, reverses_id? | 退款反向记录；不删除已支付历史 |
| reimbursement_claims | purchase_id, requested_amount, state, reviewed_by | v0.2；approved 不等于 paid |
| provider_consumption | source_scope, native_amount/unit, period, billing_semantics | 原生 credit/API metering 与现金不同 |
| price_catalog_versions | provider/model, rates, source_url, fetched_at, effective_range | 历史估算不静默变价 |
| fx_rate_versions | base/quote, decimal_rate, date, source | 不跨币种无条件合计 |
| evidence_attachments | owner, encrypted_object_ref, retention, scan_status | v0.2；首版不建上传入口 |

首版 purchases 的状态仅 draft/submitted/recorded/voided，recorded 指已登记不表示银行或供应商核验。后期审批链单独实现，不能重用 recorded 冒充 approved。

## 4. 治理实体

audit_events 记录角色变更、邀请、绑定、策略确认、报告导出、费用修改、撤销及删除，不记录请求体/凭据。使用可识别操作者的最小审计信息并受保留政策约束。

deletion_requests 保存主体、范围、原因类别、处理状态、完成时间；deletion_tombstones 在备份恢复时阻止被删数据复活，不存被删除内容。保留最少墓碑依据需由运营方确认。

retention_policies、policy_acceptances、export_jobs、notification_preferences 使用 organization_id 范围隔离。首版没有多租户商业运营承诺。

## 5. 状态机

- 员工：invited -> active -> suspended/offboarded。恢复需明确管理动作，已撤销设备凭据不能自动恢复。
- 设备：pending -> active -> revoked。设备 offline/paused 是观测/本地状态，不覆盖安全撤销。
- 授权：proposed -> active -> revoked/superseded。减少范围立即生效，增加范围重新本人确认。
- 来源：unknown -> healthy/partial/stale/auth_required/unsupported/disabled。状态不能用作员工绩效标签。
- 批次：pending local -> accepted server -> acknowledged local；冲突进入 quarantine。服务器原子提交，不存在客户端猜测成功。
- 报销（后期）：draft -> submitted -> approved/rejected -> paid；付款撤销/退款走独立事件。

## 6. 归属、去重和隔离约束

服务器从设备凭据解析组织和员工，拒绝客户端自填其他员工身份。即使请求给了 organization_id/member_id，也不作为授权依据。

默认一台 Collector 安装对应一个操作系统用户和一个组织成员。共享电脑不同系统用户分别安装/绑定；同系统用户混用员工身份不在首版支持范围。

每个报表数字须能追溯到源范围、源版本、授权版本和修订。设备A报告账号总量与设备B报告同账号总量保留两个观察但不相加。无跨设备事件ID时保留独立来源，不推断精确全员总量。

## 7. 首版建议保留期限

| 数据 | 建议默认 | 删除处理 |
|---|---|---|
| 本地待传队列 | 最长7天且最多50MiB | 超限显式标断档；授权撤销清除对应项 |
| 标准化快照/接收明细 | 90天 | 删除后保留符合政策的日聚合 |
| 日级聚合 | 13个月 | 到期删除，避免长期人员画像 |
| 订阅与购买 | 24个月 | 运营方按适用要求另行确认，不能自动当法定期限 |
| 管理审计 | 180天 | 最少字段，受限查询 |
| 备份 | 30天滚动 | 到期淘汰，恢复时重放删除墓碑 |

上表是可评审的产品默认，不是法律保留要求。离职默认撤销设备与访问，历史汇总依公示政策处理；解绑不等于立即删除所有历史，员工页面需分别提供解释和申请入口。

## 8. 数据迁移原则

实现阶段再生成版本化 SQL。不得在此阶段运行建库。未来迁移要求可备份、可验证、可回滚或提供恢复流程；字段重命名和语义改变不可无提示复用旧列。集成测试使用合成数据，不复制生产数据库。
