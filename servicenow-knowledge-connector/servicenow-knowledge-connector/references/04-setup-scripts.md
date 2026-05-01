# Set up ServiceNow Knowledge connector prerequisites by using background scripts

Source: https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/servicenow-knowledge-setup-scripts
Last updated on Microsoft Learn: 04/02/2026

You can run background scripts that automate the configuration steps for the connector. They perform the same configuration as the manual steps — no extra permissions, plugins, or external connections.

Scripts are hosted in the ServiceNow Knowledge connector setup scripts GitHub repo.

## Prerequisites

- ServiceNow admin account with `security_admin` role elevated.
- Access to **System Definition > Scripts - Background**.

## Scripts overview

| Script | What it does | Equivalent manual steps |
|---|---|---|
| `row_level_acl_setup.js` | Creates service account, custom role, and row-level READ ACLs for all required tables | Create service account and grant table access |
| `field_level_acl_setup.js` | Creates field-level READ ACLs (`table.*`) for tables where field values are restricted | Grant field-level access |
| `scripted_rest_api_setup.js` | Creates the Scripted REST API endpoint for the Advanced flow | Set up REST API |

All scripts are:
- **Idempotent** — safe to run multiple times; reuse existing records, no duplicates.
- **Non-destructive** — don't modify, delete, or overwrite existing records.
- **Self-contained** — no external dependencies or network calls.

## Step 1: Create service account and grant row-level access

1. Elevate to `security_admin`.
2. Go to **All > System Definition > Scripts - Background**.
3. Paste `row_level_acl_setup.js`.
4. Review the CONFIGURATION section (role name, user ID, user name defaults can be changed).
5. Run script.
6. Review output summary.

**What it does NOT do:**
- Grant field-level access (see Step 3).
- Set the service account password (set manually).

## Step 2: Verify row-level access

1. Set a strong unique password for the service account.
2. Query a table as the service account with Basic Auth:
   ```
   GET https://<instance>.service-now.com/api/now/table/kb_knowledge?sysparm_limit=1
   ```
3. Confirm rows are returned.

> **Note:** On Zurich and later releases, the script marks the account as machine identity (`identity_type = machine`), which auto-enables "Web service access only". These accounts can't be impersonated through the UI — use REST API to verify.

- Rows with field values populated → skip to Step 4.
- Rows but empty field values → continue to Step 3.

## Step 3: Grant field-level access (if needed)

1. Elevate to `security_admin`.
2. Paste `field_level_acl_setup.js` into Background Scripts.
3. If your role name differs from default `copilot_connector`, update `TARGET_ROLE_NAME`.
4. Adjust the `TABLES` list as needed.
5. Run script.

## Step 4: Set up REST API for advanced flow

Run if your instance uses advanced scripts in user criteria.

1. Elevate to `security_admin`.
2. Paste `scripted_rest_api_setup.js`.
3. If your service account uses a different role, update `ROLE_NAME` (default `copilot_connector`).
4. Run script. Successful run ends with:
   ```
   All steps completed successfully. No manual actions needed.
   ```
   If any step can't complete automatically (older ServiceNow versions), the output lists specific manual follow-ups.

### Verify

1. Go to **Scripted REST APIs > Microsoft Copilot**.
2. Under Security, confirm Default ACLs shows `Microsoft Copilot, Scripted REST External Default`.
3. Open the `GetAllUserCriteria` resource — confirm **Requires authentication** and **Requires ACL authorization** are checked, and ACLs shows `Microsoft Copilot, Scripted REST External Default`.
4. Note the Resource path (e.g., `/api/<namespace>/microsoft_copilot/user_criteria`). The M365 admin enters the `<namespace>` at deployment.

## Configuration options

Each script has a clearly marked CONFIGURATION section at top:

- Role name — default `copilot_connector`
- Service account user ID — default `microsoft.copilot`
- Table lists — add/remove per your instance

## Verify service account permissions

Use the Copilot Connector Checker Tool (Basic or OAuth recommended), complete fields, and **Perform Test**.
