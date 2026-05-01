# Copilot Connector Governance, Security, and Compliance — Deck Extract

Companion deck text, extracted from `governance-security-compliance.pptx` (44 slides). Use as a quick reference alongside the Microsoft Learn docs.

## Agenda

1. Fundamentals
2. Security
3. Governance
4. Compliance
5. Q&A

## Fundamentals

### Copilot Connectors — what they bring

- Ground responses with your business knowledge
- Bring tools via custom tools
- Focus user experiences with your agents
- Ref: https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/overview-copilot-connector

### Available pre-built connectors (October 2025)

Cloud / SaaS: Salesforce, Azure SQL, CSV, Oracle SQL, Media Wiki, ServiceNow, Azure DevOps, Jira Cloud, SQL Server, GitHub, Google Drive, Jira Data Center, PostgreSQL, WordPress, Zoom, Stack Overflow, Confluence Cloud/On-Prem, Enterprise Websites, Tableau Cloud, Adobe Experience Manager, Dropbox, GitLab, Asana, Miro, Veeva Vault, Smartsheet, Guru, Seismic, Aha!, Amazon S3, DataStax, 15Five Priorities, Egnyte, Zendesk, Trello, Unify, PagerDuty, Freshservice, Coda Enterprise, Shortcut, Gong, Bitbucket, BambooHR Profiles, Box, SAP SuccessFactors, Workday, Monday.com.

On-premises: File Share, ADLS Gen2, SharePoint Server (require Graph connector agent).

### Graph Connector Agent

On-premises connectors require installing the Microsoft Graph connector agent, which allows secure data transfer between on-premises data and connector APIs.


## Security

### Data flow

Large Language Model ↔ Microsoft Graph ↔ Indexed Items ← External system via connector. When reasoning over a Copilot connector, the Copilot Orchestrator does NOT call outside the Microsoft boundary — the connector pushes content into Microsoft Graph.

### Authentication concepts

- **OAuth** — authorize one app to sign in to another without exposing passwords.
- **Basic Authentication** — username/password; not recommended in production.
- **OpenID Connect with Microsoft Entra** — authentication protocol based on OAuth2 message flows.

### Authentication methods by connector

| Connector | Methods |
|---|---|
| Atlassian Jira Cloud | Basic, OAuth 2.0 (rec) |
| Azure Data Lake Storage Gen2 | Storage connection string |
| Azure DevOps (Work Items, Wiki) | OAuth |
| Azure SQL | OIDC |
| Microsoft SQL Server | Basic, Windows (agent) |
| Confluence Cloud | Basic, OAuth 2.0 (rec) |
| Confluence On-premises | Basic, OAuth1.0a, OAuth 2.0 (rec) |
| CSV | OAuth 2.0 |
| Enterprise Websites | None, Basic, OAuth 2.0 (rec) |
| File Share | Windows |
| Media Wiki | None, Basic, OAuth 2.0 (rec) |
| Oracle SQL | Basic, Windows (agent) |
| Salesforce | OAuth 2.0 |
| **ServiceNow (Catalog, Knowledge, Tickets)** | **Basic, ServiceNow OAuth 2.0 (rec), OIDC** |
| SharePoint on-prem | Basic, Windows (agent) |
| Stack Overflow | Basic, OAuth (rec) |
| Veeva Vault PromoMats | OAuth |

### Manage Search Permissions

Two options:

- **Only people with access to this data source** — only users in the source ACL see results.
- **Everyone** — everyone in the org sees results.

**Non-Microsoft Entra ID mapping** (needed for ServiceNow):

1. Select a Microsoft Entra user property — UserPrincipalName, ActiveDirectorySId, AadId, or Mail.
2. Select non-Entra user properties to map — `sys_id`, `user_name`, Email. Add a regex (e.g., `(.*)` to select full text).
3. Create a formula that combines regex outputs, e.g. `({0}SN@M365xxxx.onmicrosoft.com)`.

### Access Control Lists (ACL)

- ACLs define permissions for users/groups on indexed data.
- Specify whether roles are granted or denied.
- `accessType: deny` takes precedence over `grant`. Example: if Everyone is granted access, External Group granted, and a specific user denied — effective permission is `deny`.

### External Groups

Represent non-Entra groups (business units, teams). Needed when:
- The source uses profiles, roles, permission sets specific to the platform (e.g., Salesforce).
- Membership is not available in Entra.

### ServiceNow Knowledge — search permission behavior

- Supports **Everyone** or **Only people with access to this data source**.
- If a KB article has no user criteria, it appears in search results for everyone in the org (governed by `glide.knowman.block_access_with_no_user_criteria` default).

### Data Protection — FAQ

- **Content encryption**: encrypted with Microsoft 365 keys by default. Orgs can provide their own key. Microsoft does not have access to partner encryption keys.
- **Tenant isolation**: logical isolation, rigorous physical security, encryption at rest and in transit, compliance with applicable privacy laws.

## Governance

### What admins can govern

- Add, monitor, update, delete existing connections.
- Manage staged rollout for selective users.
- View analytics.
- Monitor items crawling and change configuration.
- Update connection and schema rules.

### Manage connections

- Monitor states: Syncing, Ready, Paused, Failed, Delete Failed.
- Be notified of crawl failures.
- Delete existing connections.

### Staged Rollout

- Deploy to a limited set of users.
- Monitor performance, adjust settings.
- Expand or shrink scope at any time.
- End rollout to deploy to the entire org.

### Monitor Item Crawling

From the Detail pane:
- Current crawl (if running)
- Last crawl (timestamp, duration, success/fail counts)
- Total item errors
- Permissions
- Items indexed
- Index limit
- Schema
- Refresh Settings

### Index Browser

- Verify indexed items (metadata, ACLs, user permissions).
- Check specific user access by entering username/email.

### View Analytics

**Crawled items** — total discovered, successfully indexed, failed across all crawls.
**Indexed items** — content, properties (default + custom), access.

### Connection Updates

Updatable (depends on connector): Name, Description, Display name, Connection icon, Crawl configuration, Schema, Refresh setting.

### Connection Analytics

Query metrics:
- Queries that included ≥1 connection result.
- Queries where user clicked ≥1 connection result.
- Number of currently active connections.
- Total indexed items across all connections.

### Connection Customization

After publication: verticals, result types, schema, relevance tuning.

### Service limitations

Categories: Connections, Schema, Group, Item Ingestion, Throttling.

## Compliance

### Data Residency and Storage

- Connector data stored in the region of the customer's M365 tenant.
- Follows same processes as 1st-party Microsoft apps.
- Connector content index is co-located with the saved content.

### European Union Data Boundary (EUDB)

- Only the EUDB provides contractual commitments for EU processing.
- Enables EU and EFTA customers to process and store customer data in the EU.
- Promises: store/process data in EU, expand EU data centers, global security, in-region support model.

### Terms of Use

- Use of Copilot connectors may be subject to third-party terms. Microsoft is not a party to those agreements.
- You are responsible for data indexing as the data controller.
- Microsoft makes no representation, warranty, or guarantee on third-party data before indexing.
- Microsoft assumes no liability for indexing errors from third-party data.

## Appendix

- Grant administrative rights to AI Administrators to manage connectors — https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/connector-admin-delegation
