---
name: servicenow-knowledge-connector
description: |
  Guide for deploying, configuring, and troubleshooting the Microsoft 365 Copilot
  ServiceNow Knowledge connector. Use when user asks to "deploy the ServiceNow
  connector", "set up ServiceNow Knowledge for Copilot", "index ServiceNow KB
  articles", "grant service account table access in ServiceNow", "run the
  background scripts for ServiceNow connector setup", "troubleshoot ServiceNow
  connector issues", "HR articles oversharing", "Advanced vs Simple flow",
  "Federated Auth for ServiceNow connector", or any ServiceNow Knowledge
  connector admin task. Do NOT use for ServiceNow Catalog or ServiceNow Tickets
  connectors — those have separate setup paths.
cowork:
  category: productivity
  icon: Book
---

# ServiceNow Knowledge Connector

Authoritative reference for the Microsoft 365 Copilot ServiceNow Knowledge connector. Content indexed from Microsoft Learn on 2026-04-24.

## When to use this skill

- Planning or executing a ServiceNow Knowledge connector deployment
- Explaining prerequisites (service account, ACLs, scoped roles, REST API)
- Helping choose Simple vs Advanced flow, or the right auth method
- Diagnosing indexing failures, missing articles, or permission issues
- Guidance on background scripts that automate ServiceNow-side setup

## Scope discipline

Ground answers in the six reference docs listed below. If asked something outside that scope (Catalog/Tickets connectors, unrelated ServiceNow admin topics, unrelated Copilot connectors), say so and offer to look it up separately. Do not invent version-specific behavior, property names, or IP ranges.

## Reference map

| Topic | File |
|---|---|
| Overview, capabilities, limitations | [01-overview](references/01-overview.md) |
| ServiceNow admin setup (tables, ACLs, REST API, hierarchical permissions) | [02-admin-setup](references/02-admin-setup.md) |
| Grant table access (user, role, row/field-level ACLs, HRSD roles) | [03-grant-table-access](references/03-grant-table-access.md) |
| Background scripts automation | [04-setup-scripts](references/04-setup-scripts.md) |
| Deployment in M365 admin center (flow, auth, defaults, schema) | [05-deployment](references/05-deployment.md) |
| Troubleshooting | [06-troubleshooting](references/06-troubleshooting.md) |

## Core decision points

**Simple vs Advanced flow** — Default is Simple. Choose Advanced only if any User Criteria uses advanced scripts. Check with:
`<ServiceNowURL>/api/now/table/user_criteria?advanced=true&sysparm_limit=1`

**Auth methods** — Federated Auth is recommended (no secret to rotate). OAuth 2.0 and Microsoft Entra ID OIDC are supported. Basic is not recommended for production.

**Hierarchical permissions** — Evaluate KB (parent) + article (child) user criteria. Requires `sys_properties` read access. Not available in government/sovereign clouds or dedicated forests.

**HRSD scope** — If KBs live in `sn_hr_core`, assign `sn_hr_core.content_reader` (minimum) or `sn_hr_core.admin` to the service account. Without it, HR-scoped user criteria silently filter and restricted HR articles can surface to everyone.

## Critical gotchas

1. **`sys_properties` access is required for both flows.** Missing it silently defaults to the most restrictive settings — articles without user criteria disappear from search with no error shown.
2. **Don't apply `snc_read_only` to the service account.** It blocks writes to `oauth_credential` needed for token refresh.
3. **`getAllUserCriteria()` is deprecated.** The troubleshooting doc has an alternative script using `sn_uc.UserCriteriaLoader.getMatchingCriteria()`.
4. **SSO "Logout successfully" loop** — fix with private browser sign-in to ServiceNow + separate M365 admin tab.
5. **OIDC "Assignment required"** — turn off in the enterprise app properties to avoid AADSTS501051.
6. **URL customization is set-once.** To change AccessURL format, create a new connection.
7. **Incremental crawl does not refresh identity.** Full crawl required after user/criteria attribute changes.

## High-level deployment flow

1. **ServiceNow admin — configure environment** (instance URL, portal URL, attribute mapping, advanced-script check).
2. **ServiceNow admin — prerequisites** (service account, row/field ACLs, REST API for Advanced, `sys_properties` access, HRSD roles, IP allowlist).
   - Manual path: [03-grant-table-access](references/03-grant-table-access.md)
   - Automated path: [04-setup-scripts](references/04-setup-scripts.md)
3. **Verify** with the Copilot Connector Checker Tool.
4. **M365 admin — deploy** via M365 admin center > Copilot > Connectors > Gallery > ServiceNow Knowledge. Pick flow, URL, auth, API namespace (if Advanced), staged rollout.
5. **Validate** indexed count against the expected `sys_id` count. When status is Ready, test with a known article's `sys_id`.

## How to answer common asks

- **"What permissions does the service account need?"** → Point to the table in [02-admin-setup](references/02-admin-setup.md) and call out Simple-only tables (marked **) and HRSD-scope requirements.
- **"Articles missing from search"** → Walk through the troubleshooting steps in order: user permissions → Entra mapping (2006 error) → advanced scripts → `sys_properties` access → HRSD scope → Access Analyzer.
- **"What do the background scripts do?"** → Three scripts: `row_level_acl_setup.js` (user + role + row ACLs), `field_level_acl_setup.js` (field-level ACLs), `scripted_rest_api_setup.js` (REST API for Advanced flow). Idempotent and non-destructive.
- **"Which auth method should we use?"** → Federated Auth if their ServiceNow version supports OIDC inbound integrations. OAuth 2.0 is the tried-and-true fallback. Never Basic for production.
- **"Indexed item count check"** → Use the console fetch snippet against `/api/now/table/kb_knowledge?sysparm_fields=sys_id&workflow_state=published` and compare to the connector's reported count.
