# Set up the ServiceNow service for ServiceNow Knowledge connector ingestion

Source: https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/servicenow-knowledge-admin-setup
Last updated on Microsoft Learn: 04/08/2026

This article provides information about the configuration steps that ServiceNow admins must complete before your organization deploys the ServiceNow Knowledge connector.

## Setup checklist

### Configure the environment

| Task | Role |
|---|---|
| Identify the instance URL | ServiceNow admin |
| Identify the portal configuration | ServiceNow admin |
| Define attribute mapping | ServiceNow admin |
| Check for advanced scripts and hierarchical permissions | ServiceNow admin |

### Set up prerequisites

| Task | Role |
|---|---|
| Create service account and set up permissions | ServiceNow admin |
| Verify service account permissions | ServiceNow admin |
| Identify item count for ingestion | ServiceNow admin |
| Set up REST API | ServiceNow admin |
| Set up hierarchical permissions | ServiceNow admin |
| Add Microsoft 365 IP address to allowlist | ServiceNow admin / Network admin |
| Resolve issues with SSO configuration | ServiceNow admin |

## Configure the ServiceNow environment

### Identify the ServiceNow instance URL

Typical format: `https://<your-organization-name>.service-now.com`

For custom URLs: In ServiceNow, go to **All > Custom URL > Custom URLs**.

### Identify the ServiceNow portal configuration

Default link format used by Copilot: `https://<your-organization-name>.service-now.com/kb_view.do?sys_kb_id=...`. If your org uses a different URL, customize it at deploy time (see Customize values for certain schema properties).

### Define ServiceNow attribute mapping

By default, Microsoft Entra ID maps identities from ServiceNow by matching email to UPN or Mail. For a custom mapping, see Map your non-Entra ID identities.

### Check for advanced scripts and hierarchical permissions

Detect whether knowledge articles use:

- Advanced scripts enabled in User Criteria
- Hierarchical permissions

To check for advanced scripts, call: `<ServiceNowURL>/api/now/table/user_criteria?advanced=true&sysparm_limit=1`

If present, choose **Advanced flow** at deployment. Hierarchical permissions support KB-level (parent) and article-level (child) evaluation — not available in government or sovereign clouds or dedicated forests in multitenant environments.

## Set up connector prerequisites

> **Tip:** You can automate the prerequisite setup steps by using background scripts instead of the manual steps in this article. See the setup scripts doc.

### Create service account and set up permissions to index items

Required table records:

| Feature | Table | Description |
|---|---|---|
| Index KB articles (Everyone) | `kb_knowledge` | Crawling knowledge articles |
| Extended table properties | `sys_db_object` | Extended table details for templates |
|  | `sys_dictionary` | Extended table properties and crawl templates |
|  | `sys_properties` | System config properties to evaluate article permissions (required for Simple and Advanced) |
|  | `sys_attachment` | Crawling attachments |
|  | `kb_feedback` | Crawling comments |
| User criteria permissions | `kb_uc_can_read_mtom` | Who can read this KB |
|  | `kb_uc_can_contribute_mtom` | Who can contribute to this KB |
|  | `kb_uc_cannot_read_mtom` | Who can't read this KB |
|  | `kb_uc_cannot_contribute_mtom` | Who can't contribute to this KB |
|  | `sys_user` | Read user table |
|  | `sys_user_has_role` | Read role info of users |
|  | `sys_user_grmember`** | Read group membership |
|  | `user_criteria` | Read user criteria permissions |
|  | `kb_knowledge_base` | Read knowledge base info |
|  | `sys_user_group` | Read user group segments |
|  | `sys_user_role` | Read user roles |
|  | `cmn_location`** | Read location info |
|  | `cmn_department`** | Read department info |
|  | `core_company`** | Read company attributes |

** Only required for Simple flow.

Optional roles (to avoid ACL blocking issues): `knowledge_admin`, `user_criteria_admin`, `user_admin`.

**Note:** Don't apply `snc_read_only` to the service account — it needs write access to `oauth_credential` for token refresh.

**Important:** `sys_properties` is required for both Simple and Advanced flows. The connector reads `glide.knowman.apply_article_read_criteria` and `glide.knowman.block_access_with_no_user_criteria`. Without access, the connector silently defaults to the most restrictive settings (articles without explicit user criteria get blocked from search). No error is shown.

If the service account doesn't have full User Criteria table access, inconsistent permissions behavior can occur — including unintended oversharing, especially for HRSD content.

### Additional roles for HR Service Delivery (HRSD) content

If KBs are in the `sn_hr_core` application scope, assign one of:

| Role | Description |
|---|---|
| `sn_hr_core.content_reader` | Satisfies the HR Core-scoped ACL on `user_criteria`. Minimum-privilege option. |
| `sn_hr_core.admin` | Scope-level admin access to all data in HR Core scope, including user criteria. |

These roles are independent — `sn_hr_core.admin` does NOT contain `sn_hr_core.content_reader` in the default hierarchy.

**Warning:** Without one of these, the service account can query `user_criteria` through the global-scope ACL, but HR-scoped rows are silently filtered. The connector receives empty results rather than an error, and might treat HR articles as accessible to everyone — risking exposure of compensation, benefits, or disciplinary content.

### Verify service account permissions

Use the Copilot Connector Checker Tool. Choose Basic or OAuth (recommended), complete the fields, and choose **Perform Test**.

### Identify item count for ingestion

Default filter: `active=true^workflow_state=published`

1. Go to `https://<instance-name>.service-now.com/api/now/table/kb_knowledge?sysparm_fields=sys_id&workflow_state=published`
2. In Developer Tools console:
   ```js
   fetch('<<same URL as in 1>>')
     .then(res => res.json())
     .then(data => console.log("Count of sys_ids:", data.result.length))
     .catch(err => console.error(err));
   ```
3. Record the count and compare after indexing.

### Set up REST API

Required if using Advanced flow.

1. Elevate to `security_admin`.
2. Create ACL at **All > System Security > Access Control (ACL)** with:
   - Type: `REST_Endpoint`, Operation: `Execute`, Name: `Microsoft Copilot`, Role: `admin` (or your crawler role).
3. Create Scripted REST API at **All > System Web Services > Scripted Web Services > Scripted REST APIs**:
   - Name: `Microsoft Copilot`, ID: `microsoft_copilot`.
   - Default ACLs: `Microsoft Copilot` + `Scripted REST External Default ACL`.
4. Add resource on Resources tab:
   - Name: `GetAllUserCriteria`, Relative Path: `/user_criteria`.
   - Script:
     ```js
     (function execute(/*RESTAPIRequest*/ request, /*RESTAPIResponse*/ response) {
         var queryParams = request.queryParams;
         var user = new String(queryParams.user);
         return (new sn_uc.UserCriteriaLoader()).getAllUserCriteria(user);
     })(request, response);
     ```
   - Check: **Requires authentication** and **Requires ACL authorization**.
   - ACLs: `Microsoft Copilot` + `Scripted REST External Default ACL`.
5. Resource path: `/api/<API Namespace>/microsoft_copilot/user_criteria`. The M365 admin enters the namespace at deployment.

> **Note:** `getAllUserCriteria()` is deprecated due to performance. An alternative script is in the troubleshooting doc.

### Set up hierarchical permissions

Grant the service account read access to `sys_properties`. The connector reads:

- `glide.knowman.apply_article_read_criteria`
- `glide.knowman.block_access_with_no_user_criteria`

If you want to restrict the service account to only the required property fields, create three ACLs on `sys_properties`:

1. **Row-level** (Name as `--None--`): Allow read where Name is `glide.knowman.apply_article_read_criteria` OR `glide.knowman.block_access_with_no_user_criteria`.
2. **Field-level on Name**: Same predicate, field set to Name.
3. **Field-level on Value**: Same predicate, field set to Value.

Each ACL: Type `record`, Application `Global`, Active checked, Decision Type `Allow`, Admin overrides checked, Operation `read`, and Requires Roles = your service account role.

### Add Microsoft 365 IP address to the allowlist

If firewall/proxy blocks access, allowlist the IPs in the troubleshooting article. For ServiceNow-specific controls, see IP Address Access Control.

### Resolve connector setup issues with ServiceNow SSO configuration

Symptoms:
- "Logout successfully" window during OAuth without a credential prompt.
- M365 admin credentials used instead of the service account.

Fix:
1. Open a private browser window and sign in with ServiceNow service account credentials.
2. In a new tab, sign in to M365 admin center with M365 admin credentials.
3. Retry OAuth — you should now be prompted to authorize with the service account.
