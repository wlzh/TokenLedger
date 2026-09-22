# TokenLedger

[简体中文](README.zh-CN.md) | [Documentation map](docs/README.md)

Employee-authorized AI subscription, usage, and expense reporting for small teams.

**Status: documentation-only design draft. No agent, server, web app, installer, or hosted service has been implemented.** TokenLedger is the confirmed project name; public name/trademark availability has not been verified.

Maintainer and project lead: [@wlzh](https://github.com/wlzh). Public documentation repository: [wlzh/TokenLedger](https://github.com/wlzh/TokenLedger).

## Platforms and upstream acknowledgments

TokenLedger is planned as a team reporting layer built on two personal AI usage tools, not a replacement for them:

| Employee platform | Upstream tool | Planned integration |
|---|---|---|
| macOS | [CodexBar](https://github.com/steipete/CodexBar) by Peter Steinberger and contributors | Independent collector reading supported local CLI outputs |
| Windows | [Win-CodexBar](https://github.com/nesszer/Win-CodexBar) by nesszer and contributors | Separate Windows adapter reading supported local CLI outputs |

Thank you to both projects and their contributors for their open-source work on personal AI usage visibility. Their provider integrations and local reporting capabilities make this proposed team layer possible. Employees would keep their existing upstream tools and accounts; TokenLedger would add employee-authorized reporting, subscriptions, expenses, and a company dashboard.

Both platforms are design targets, not tested product support. Upstream installation alone does not send data to TokenLedger; a separately installed and explicitly authorized collector is required. No upstream source or binaries are bundled in this documentation-only repository. See [compatibility](docs/compatibility.md) and [third-party notices](THIRD_PARTY_NOTICES.md). This is an independent project without upstream endorsement.

## Intended workflow

Employees keep their own AI subscriptions and use CodexBar on macOS or Win-CodexBar on Windows. An independently installed, opt-in collector would normalize supported local outputs, remove sensitive fields, and send approved metrics to a company-operated HTTPS server. Managers would review spending and usage by employee, product, account, and reporting period.

The project does not require employees to share provider credentials or replace subscriptions with a company API gateway.

## Planned capabilities

- Employee invitations, device enrollment, selected account reporting, and self-service visibility.
- Provider quota snapshots and supported local token summaries with explicit provenance and coverage.
- Subscription and purchase registration, separating cash payments, provider-metered consumption, and list-price estimates.
- Authenticated, self-hosted dashboards with employee-scoped access and organization-level administration.
- Offline buffering, idempotent ingestion, account attribution controls, and data-health reporting.
- Later phases: evidence-backed expense approvals, activity estimates, notifications, and additional integrations.

## What this will not claim

- A quota percentage is not a token count.
- A local API-price estimate is not an invoice or subscription charge.
- Logged-in account identity does not establish ownership of historical logs.
- Local logs do not cover every device, browser session, mobile app, or cloud task.
- AI activity is not employee working time or productivity.
- Endpoint reports are not tamper-proof provider attestations.

## Design and verification status

Source inspection used CodexBar commit `b99a91694d7878202ee0b6d0bc725952bbd8c328` and Win-CodexBar commit `8cdd2cad1f52102ad314f3bad6d8ca6ec2c8a056`. This is not a runtime compatibility claim. See [upstream evidence](docs/upstream-audit.md) and the [compatibility matrix](docs/compatibility.md).

There is no installation command yet. Do not run hypothetical commands, expose upstream local HTTP servers publicly, or submit real credentials to sample payloads. All examples use synthetic identifiers and `.invalid` domains.

## Read first

1. [Executive proposal](docs/overview.md)
2. [Product requirements](docs/prd/v0.1.0/prd.md)
3. [Interaction design](docs/prd/v0.1.0/design.md)
4. [Technical design](docs/prd/v0.1.0/dev.md)
5. [Metrics definitions](docs/metrics.md)
6. [Milestones and approval gates](docs/prd/v0.1.0/plan.md)

## Privacy and security

The design permits selected usage metadata to leave the employee device for the configured organization server. It does **not** copy upstream claims of zero data collection. Provider API keys, cookies, OAuth tokens, prompts, responses, source code, session titles, and project paths must not be uploaded.

See [privacy design](PRIVACY.md), [security policy](SECURITY.md), [threat model](docs/security-threat-model.md), and [employee reporting notice](docs/employee-notice.md). These are design requirements, not implemented guarantees. Operators must identify their own contact, jurisdiction, hosting location, retention, and employment-policy requirements before a pilot.

## Licensing and attribution

New project material is provided under the [MIT License](LICENSE). This is an independent project, not an official CodexBar, Win-CodexBar, OpenAI, or Anthropic product. See [third-party notices](THIRD_PARTY_NOTICES.md). Referencing or invoking an upstream project does not grant rights to provider services or trademarks.

The project is maintained by @wlzh at `wlzh/TokenLedger`. A private security channel, signing identities, package names, and production domains still need configuration before software distribution or operation. Contributions follow [CONTRIBUTING.md](CONTRIBUTING.md).
