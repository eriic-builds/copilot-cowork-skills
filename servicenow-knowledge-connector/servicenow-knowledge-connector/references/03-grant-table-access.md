# Grant table access to a service account in ServiceNow Knowledge

Source: https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/granting-table-access-servicenow-knowledge
Last updated on Microsoft Learn: 04/08/2026

This article explains how to grant table access to a service account in ServiceNow Knowledge: create a role, assign to a user, and configure row-level and field-level access controls.

> **Tip:** Instead of manual steps, run background scripts (`row_level_acl_setup.js`, `field_level_acl_setup.js`) that create the user, role, and ACLs automatically.

## Prerequisites

- Admin access in ServiceNow.
- Elevate to `security_admin` role to modify ACLs.

## Create a user

1. Go to **User Administration > Users**.
2. Select **New**.
3. Fill in:
   - User ID: `microsoft.copilot` (required for successful crawls)
   - First/Last Name: `Microsoft` / `Copilot`
   - Identity Type: `Machine`
   - For earlier ServiceNow versions, check **Web service access only**.
4. Submit.

## Create a role

1. Go to **User Administration > Roles**.
2. Select **New**.
3. Name it, for example, `Copilot Connector Account`.
4. Submit.

## Assign the role to a user

1. Go to **User Administration > Users**.
2. Open the user record (e.g., `Microsoft Copilot`).
3. In **Roles** related list, select **Edit** and add the new role.
4. Save, then Update.

**Optional additional roles** (to avoid ACL blocking): `knowledge_admin`, `user_criteria_admin`, `user_admin`.

**HRSD:** If your instance uses HR Service Delivery KBs in `sn_hr_core`, also assign `sn_hr_core.content_reader` (or `sn_hr_core.admin` for broader access). Without one, HR-scoped user criteria are silently filtered, risking oversharing.

## Grant row-level access

1. Elevate to `security_admin`.
2. **System Security > Access Control (ACL) > New**.
3. Fill in:
   - Type: `record`
   - Operation: `read`
   - Name: the table (e.g., `sys_dictionary`)
   - Roles: add your `Copilot Connector Account`.
4. Submit.

### Verification

Impersonate the user and access the table. If rows are visible but values are empty, grant field-level access.

## Grant field-level access

1. **System Security > Access Control (ACL) > New**.
2. Fill in:
   - Type: `record`
   - Operation: `read`
   - Name: the table, use `*` in the field name (applies to all fields)
   - Roles: add `Copilot Connector Account`.
3. Submit.

### Final verification

Impersonate the user and confirm both rows and field values are visible.

## Verify service account permissions

Use the Copilot Connector Checker Tool (Basic or OAuth auth), complete the fields, and **Perform Test**.
