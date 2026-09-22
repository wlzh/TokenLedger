# Security Policy

## Current status

This repository is a documentation-only design draft. There are no released executables, hosted services, or supported runtime versions. Security requirements are described in [the threat model](docs/security-threat-model.md), not implemented assurances.

## Reporting

Project maintainer and security coordination owner: [@wlzh](https://github.com/wlzh). Repository: [wlzh/TokenLedger](https://github.com/wlzh/TokenLedger). A GitHub profile is not a private disclosure channel; no email address or enabled private vulnerability reporting feature is assumed.

Do not publish provider credentials, employee records, raw logs, database exports, or exploitable production details in public issues. A private maintainer contact and disclosure workflow must be configured before any pilot or public software release. No security email address or response SLA has been established yet.

Until then, communicate privately with the project owner through an already established channel. If no private channel is available, request one without posting sensitive details. The operator of a deployed instance is responsible for its incident response and employee notifications.

Reports should include affected version, sanitized reproduction, impact, and whether real data may be involved. Future fixes require regression tests, supported-version decisions, release notes, and coordinated disclosure where appropriate.
