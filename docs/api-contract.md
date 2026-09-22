# 接口与上报契约

状态：设计草案 `ingest/1`，所有路径仅为拟议接口，没有已运行端点。

## 1. 身份与传输

公网仅 HTTPS。员工浏览器使用短会话、HttpOnly/Secure/SameSite Cookie，变更接口需要 CSRF 防护。Collector 使用独立设备 bearer 凭据，保存在系统凭据库，服务器只存摘要；不得使用供应商 API Key 认证本系统。

设备仅有写入本成员数据与读取本设备最小状态的权限，没有读取全公司报表、改变角色、购买产品、远程执行权限。管理员凭据不得分发到员工设备。

## 2. 首版接口影响清单

| 方法与路径 | 调用方 | 用途/授权边界 |
|---|---|---|
| POST /api/v1/invitations | Owner/Admin | 创建绑定目标身份的一次性邀请 |
| POST /api/v1/invitations/accept | 已验证目标身份 | 原子接受邀请，绑定成员 |
| POST /api/v1/device-enrollments | Collector | 申请短期配对事务，无业务数据上传 |
| POST /api/v1/device-enrollments/{id}/approve | 当前员工浏览器 | 确认设备、组织、服务端域名 |
| POST /api/v1/device-enrollments/{id}/exchange | 持配对秘密的Collector | 单次换取设备凭据；代码本身不足以兑换 |
| GET /api/v1/device/status | 设备 | 当前凭据/授权状态；不返回任意命令 |
| POST /api/v1/consents | 员工浏览器 | 确认选择范围、字段组、目的地 |
| POST /api/v1/consents/{id}/revoke | 本员工 | 撤销上传授权，旧批次失效 |
| POST /api/v1/ingest/batches | 设备 | 幂等原子接收已批准白名单数据 |
| POST /api/v1/device-credentials/rotate | 当前有效设备 | 换取新凭据，限制短暂交接窗口 |
| DELETE /api/v1/devices/{id} | 本员工或管理员 | 撤销设备，不能代员工新增采集范围 |
| GET /api/v1/me/usage | 员工 | 只看本人授权范围 |
| GET /api/v1/members/{id}/usage | 管理员 | 本组织成员统计 |
| GET /api/v1/overview | 管理员 | 组织汇总及覆盖情况 |
| GET/POST/PATCH /api/v1/subscriptions | 按角色 | 订阅登记，员工仅本人 |
| GET/POST/PATCH /api/v1/purchases | 按角色 | 申报与购买记录，保留修改审计 |
| POST /api/v1/exports | 按角色 | 根据同一查询权限生成受限报告 |
| POST /api/v1/deletion-requests | 本人/授权管理员 | 申请删除，返回政策和进度 |

附件、SSO、外发 webhook 与远程控制不在首版接口中。数据查询默认分页，服务端限定日期范围与导出规模。

## 3. 配对设计

Collector 生成本地配对秘密，服务器返回一个短期 device_code 与 user_code。员工在明确展示公司域名的网页确认；仅拥有短 user_code 不足以取得设备凭据。配对有效期建议10分钟、每设备请求限速、失败次数受限，配对成功即作废。

邀请接受需绑定预期目标身份；管理员从已有安全通道确认员工。首次 Owner 引导凭据只在部署主机交互输出一次，不写镜像、公开文档或日志。

## 4. 批次信封示例

以下全部为合成数据，示例为日汇总修订而不是原始日志。十进制字符串用于整数计数，避免 JS 数值精度损失。

```json
{
  "schema_version": "ingest/1",
  "batch_id": "batch_demo_0001",
  "stream_id": "stream_demo_01",
  "sequence": "17",
  "device_id": "device_demo_mac",
  "consent_id": "consent_demo_03",
  "agent_version": "0.1.0-design",
  "sent_at": "2026-09-22T09:00:00Z",
  "records": [
    {
      "kind": "usage_summary_revision",
      "record_key": "summary_demo_01",
      "revision": "2",
      "source_scope_id": "scope_demo_codex_native",
      "provider": "codex",
      "product": "codex",
      "provider_account_id": null,
      "attribution": "unassigned",
      "source_kind": "upstream_summary",
      "source_scope": "device_log",
      "measurement_kind": "observed",
      "verification": "client_reported",
      "source_version": "pinned-fixture-1",
      "adapter_version": "mac/1",
      "period_start": "2026-09-21T16:00:00Z",
      "period_end": "2026-09-22T16:00:00Z",
      "bucket_timezone": "Asia/Shanghai",
      "source_updated_at": "2026-09-22T08:59:00Z",
      "coverage_through": "2026-09-22T08:58:00Z",
      "coverage": "partial",
      "token_semantics": "normalized/1",
      "input_tokens_including_cache": "1000",
      "cache_read_tokens": "600",
      "cache_write_tokens": null,
      "output_tokens_including_reasoning": "200",
      "reasoning_tokens": "80",
      "total_tokens": "1200",
      "estimated_cost": null,
      "unavailable_reason": "price_unknown"
    }
  ]
}
```

示例的 `unassigned` 只有在授权包含“该已选日志范围的员工级汇总”时可接受，不能借此绕过账号选择。`coverage_through` 允许当日桶尚未结束，未来覆盖不被虚构。

服务端认可 provider/source/adapter 的组合须来自兼容注册表，不能因客户端写一个名字就信任语义。不存在的供应商可登记为手工订阅，但不得创建假自动用量。

## 5. 数据类型和验证

- 时间为有 UTC offset 的 RFC3339；持久化 UTC，源时区另存 IANA 名称。
- 金额和计数用有限长度十进制字符串，禁止 NaN、Infinity、负 token、溢出和不规范格式。
- 来源空值用 null，不以缺字段推断0。指标单位必需且与 kind 匹配。
- 比例值是 0..1；source window 无实际数值时不产生假额度。
- 模型 ID 是受长限的命名空间标识，不接收任意自由文本描述。
- 业务 alias 来自员工明确设置，不把会话标题当 alias。
- record_kind 严格字段白名单，未知字段拒绝；递归字段和总大小受限。
- 每批建议最多500记录、未压缩体最多1MiB；首版拒绝压缩请求以减少解压攻击面。
- 不保存验证失败原请求体，日志只存 reason_code、request_id 和有限标识。
- source_updated_at 超出服务端未来5分钟拒绝/隔离；旧记录可在授权与回填范围内入账，按发生时间归档。

## 6. 幂等、乱序和失败处理

幂等键为设备身份 + stream_id + batch_id。服务器以确定性编码计算内容摘要，不信任客户端自称摘要。

1. 同键同摘要已完成，返回同一 acceptance，不重复入账。
2. 同键不同摘要返回409，客户端隔离并提示冲突。
3. 批次原子校验与提交；不部分接收后让客户端猜哪些成功。
4. 来源 revision 决定同范围新旧；网络发送 sequence 只诊断断档，不能拒绝所有乱序补传。
5. 同 key/revision 内容冲突拒绝；较旧 revision 不覆盖新值，响应明确 skipped_stale 数量。
6. ACK 丢失时客户端用同 batch 重试；401/403 不重试洪泛，转需处理状态；429 遵守 Retry-After。
7. 无效记录导致整批422；客户端隔离该批，修正/拆分后用新 batch_id，原始记录仍有追踪链。
8. 跨设备镜像日志无法以批次幂等解决，按指标文档独立处理。

接受响应示例：

```json
{
  "batch_id": "batch_demo_0001",
  "status": "accepted",
  "accepted_records": 1,
  "skipped_stale_records": 0,
  "received_at": "2026-09-22T09:00:02Z",
  "next_upload_after_seconds": 300
}
```

响应只允许有限调度建议与状态，不允许 shell、文件路径、下载执行、供应商凭据请求。客户端必须忽略/拒绝未知控制字段。

## 7. 错误代码

| HTTP | code | 动作 |
|---|---|---|
| 400 | malformed_envelope | 隔离，显示安全诊断 |
| 401 | device_auth_required | 停止自动上传，重新配对/续期 |
| 403 | consent_revoked / device_revoked / scope_denied | 清理不再授权的待传项，不自动重授权 |
| 409 | idempotency_conflict / revision_conflict | 保留冲突状态，不修改既有结果 |
| 413 | batch_too_large | 本地拆分，新batch，保持记录归属 |
| 422 | record_invalid / unsupported_schema | 隔离，无原始数据回显 |
| 429 | rate_limited | 退避并加入随机抖动 |
| 503 | temporarily_unavailable | 限时重试，保留队列 |

## 8. 撤销与授权时间

服务器提交事务时重新检查设备及当前授权。撤销之前已经完成提交的数据不自动消失，走独立删除流程。暂停后的客户端不得开始新的请求；在途请求尽力取消，已提交结果通过员工页面说明。

授权范围缩减后，队列中超出新范围的数据丢弃并记录非内容计数。目的地域名/组织改变必须重新配对与确认；禁止后台静默重定向含认证请求至其他主机。

## 9. 版本与演进

schema major 不兼容时明确拒绝；兼容客户端版本通过测试声明，不按版本号猜测。新增可选字段也要更新端到端白名单与 fixture 测试后才允许上传。

外部 JSON 字段更新先进入适配器，不直接改变上报协议。查询 API 与 ingest 协议分开演进。未来签名/远程证明若引入，需新威胁评审；不把当前 bearer 接入称为硬件级可信计量。
