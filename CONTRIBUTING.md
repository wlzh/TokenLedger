# Contributing

TokenLedger is maintained by [@wlzh](https://github.com/wlzh) at [wlzh/TokenLedger](https://github.com/wlzh/TokenLedger). Project-wide reviews are coordinated by the maintainer; deployment-specific employee policies remain the responsibility of each operating company.

The project is currently in documentation review. There is no build, development server, or test runner to execute. Do not introduce implementation as though the design had already been approved.

## Review workflow

Propose changes to requirements, metrics, API contracts, privacy boundaries, or compatibility evidence in the relevant document. Explain the problem, tradeoff, and acceptance case. Keep the [decision log](docs/decisions.md) and version documents consistent. Implementation follows the explicit approval gates in the plan.

## Data and attribution

Use synthetic fixtures. Never commit AI credentials, cookies, real employee emails, billing receipts, prompts, code from private conversations, session titles, or local project paths. Sanitization must happen before sharing, not after a public commit. Screenshots also require inspection.

Credit copied material and retain licenses. Contributors must have the right to submit their work under MIT. No CLA or DCO automation is currently established; any future change requires a documented governance decision.

## Future code review requirements

Changes to accounting need numerical tests; adapters need versioned fixtures and null/coverage semantics; authorization needs negative tests; migrations need upgrade/rollback plans; UI needs empty/error/stale/permission states. A passing build alone is insufficient. Keep docs and release support matrices synchronized.
