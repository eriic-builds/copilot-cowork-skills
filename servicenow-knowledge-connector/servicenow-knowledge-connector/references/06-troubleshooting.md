# Troubleshoot issues with the ServiceNow Knowledge Copilot connector

Source: https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/servicenow-knowledge-troubleshooting
Last updated on Microsoft Learn: 04/08/2026

## Can't find ServiceNow Knowledge articles in Copilot or Microsoft Search

Steps:

1. Check the user has required permissions.
2. Check the user is mapped to a Microsoft Entra identity. Mapping problems show as a 2006 error on the Error tab.
3. Check whether an advanced script in any user criteria grants access — advanced scripts require the Advanced flow and REST API.
4. Use **User criteria diagnostics tool** in ServiceNow.
5. Use **Access Analyzer tool** to debug missing table/field/record access.
6. Grant required roles and ACLs.

If you can't identify root cause, contact Copilot connector support with:
- Tenant ID
- Connection ID
- Article Sys ID
- Knowledge base Sys ID
- For KB: user criteria sys_ids from `kb_uc_can_read_mtom`, `kb_uc_cannot_read_mtom`, `kb_uc_cannot_contribute_mtom`, `kb_uc_can_contribute_mtom`.
- For item sys_id: user criteria sys_ids in `can_read_user_criteria` and `cannot_read_user_criteria` of the article.

After providing access, start a **full crawl**.

## Missing access to certain tables

Must be ServiceNow admin.

### Validate via REST API Explorer

1. Impersonate the crawling account.
2. Ensure roles `rest_api_explorer` and `web_service_admin`.
3. **System Web Services > REST > REST API Explorer**.
4. Select the table from the error message.
5. Set `sysparm_limit=10`. Send.
6. Review:
   - **403** → not authorized; grant table-level access.
   - **200 with empty fields** → row access OK, field-level access missing.
   - No table in dropdown → no access at all.

### Validate via browser

Private window → `https://<instance-url>/api/now/table/<table_name>?sysparm_limit=10` with crawling-account credentials.

After granting access, start a full crawl.

## Articles without user criteria don't appear in search results

Root cause: service account lacks `sys_properties` read access. The connector reads:

- `glide.knowman.apply_article_read_criteria`
- `glide.knowman.block_access_with_no_user_criteria`

Without access, the connector defaults to the most restrictive settings. No error shown.

**Fix:** Grant read on `sys_properties` and start a full crawl. Applies to both Simple and Advanced flows.

## HR knowledge articles visible to unintended audience

### Cause

`sn_hr_core` scope enforces a separate scoped ACL on `user_criteria`. Requires `sn_hr_core.content_reader` or `sn_hr_core.admin`. Without either, HR-scoped rows are silently filtered — connector receives empty results and may default to granting access to all.

### Resolution

1. Assign `sn_hr_core.content_reader` (minimum) or `sn_hr_core.admin` (broader).
2. Verify:
   ```
   https://<instance-name>.service-now.com/api/now/table/user_criteria?sysparm_limit=10
   ```
   HR-scoped user criteria should be present.
3. Start a full crawl.

**Note:** `sn_hr_core.admin` does NOT contain `sn_hr_core.content_reader`. Both independently satisfy the ACL, through different mechanisms.

## Issue reading all user criteria from ServiceNow

Happens when `gs.getUserId()` or `gs.getUser()` are used in user criteria. ServiceNow recommends `user_id`. Remove these calls.

If performance issues with `getAllUserCriteria()` (deprecated), use this alternative script when setting up REST API:

```javascript
(function execute(/*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
    var queryParams = request.queryParams;
    var userSysId = queryParams.user ? String(queryParams.user) : null;
    var result = [];

    if (!userSysId) {
        gs.warn("UserCriteriaLoader API: 'user' parameter was not provided in the request.");
        response.setStatus(400);
        return { "error": "User sys_id is required." };
    }

    try {
        var userCriteriaLoader = new sn_uc.UserCriteriaLoader();
        var userCriterias = [];
        var userCriteriaGr = new GlideRecord('user_criteria');
        userCriteriaGr.addQuery('active', true);
        userCriteriaGr.query();
        while (userCriteriaGr.next()) {
            userCriterias.push(userCriteriaGr.getUniqueValue());
        }
        var matchingCriteriaIds = sn_uc.UserCriteriaLoader.getMatchingCriteria(userSysId, userCriterias);
        return matchingCriteriaIds;
    } catch (e) {
        gs.error("UserCriteriaLoader API: Error processing user criteria for user " + userSysId + ". Error: " + e.message);
        response.setStatus(500);
        return { error_message: "Error processing user criteria for user " + userSysId, error_details: e.message };
    }
})(request, response);
```

## Unable to sign in due to single sign-on enabled ServiceNow instance

Bring up username/password auth by appending `login.do`:
`https://<your-organization-domain>.service-now.com/login.do`

## Can't connect with the ServiceNow instance

Forbidden/Unauthorized causes:

- **Incorrect password** — Basic: verify credentials. OAuth 2.0: password wasn't reset (tokens refresh every 12 hours; reauthenticate if changed).
- **Table access permissions** — verify service account has read on all required tables.
- **Firewall** — allowlist the following connector service IPs:

| Environment | Region | Range |
|---|---|---|
| PROD | North America | `52.250.92.252/30`, `52.224.250.216/30` |
| PROD | Europe | `20.54.41.208/30`, `51.105.159.88/30` |
| PROD | Asia Pacific | `52.139.188.212/30`, `20.43.146.44/30` |

## Change the URL of the knowledge article

Customizable at initial setup only. Existing connection → create a new one to customize the URL.

## Issues with "Only people with access to this data source" permission

### Can't select the option

Service account lacks read permissions to user criteria tables.

## User mapping failures

ServiceNow users without a corresponding Entra user fail mapping. Service/nonuser accounts are expected to fail. View failures in identity stats; download logs from Error tab.

## "Logout successful" window appears during OAuth

By default, ServiceNow tries SSO with M365 admin creds.

**Resolve:**
1. Private window → sign in with ServiceNow credentials.
2. New tab → sign in to M365 admin center.
3. Retry OAuth — auth prompt should appear.

## Issue with OIDC-based authorization

Tight security settings that enable **Assignment required** on the enterprise app yield:

```
AADSTS501051: Application '<AppID>' is not assigned to a role for the application '<AppID>'.
```

**Fix:** Entra admin center > Enterprise Apps > All apps > your OIDC app > Properties > turn off **Assignment required** > Save.
