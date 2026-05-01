# ServiceNow Knowledge connector overview

Source: https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/servicenow-knowledge-overview
Last updated on Microsoft Learn: 03/19/2026

The ServiceNow Knowledge Microsoft 365 Copilot connector enables organizations to index ServiceNow knowledge base (KB) articles into Microsoft 365 Copilot and search experiences. When you index ServiceNow knowledge base content, users can retrieve relevant information using natural language queries across Microsoft 365 apps. This connector enhances productivity by surfacing IT service management data directly in Copilot, streamlining workflows and improving decision-making.

## Why use the ServiceNow Knowledge connector to index your data?

The ServiceNow Knowledge connector is ideal for organizations that rely on ServiceNow for IT operations, HR services, and customer support. Organizations can use the connector to:

- Enable employees to query ticket status, incident history, and service requests directly from Copilot.
- Allow HR teams to access onboarding workflows and policy documents.
- Empower support agents to retrieve troubleshooting steps and resolution logs.

This connector helps reduce context switching and improves access to operational data across departments.

### Use cases

Enable your users to ask questions related to your IT/HR workflows in Copilot, such as:

- How can I request a new device?
- How do I create a new VPN connection?
- How do I apply for leaves?

## Build agents with the ServiceNow Knowledge connector

Developers can use the ServiceNow Knowledge connector as a knowledge source in declarative agents built with Microsoft Copilot Studio, Agent Builder in Microsoft 365 Copilot, or the Microsoft 365 Agents Toolkit.

### Example prompts

| Department | Role | Example Prompt |
|---|---|---|
| IT | Support Analyst | "Show me all open incidents assigned to me this week." |
| HR | Manager | "What is the onboarding status for new hires in September?" |
| Operations | Admin | "List all service requests pending approval." |

## Connector capabilities and limitations

### Capabilities

- Indexes all types of knowledge articles.
- Enables Copilot and search experiences in Microsoft 365 to respond to user questions related to your IT/HR workflows.
- Uses semantic search in Copilot to enable users to find relevant content based on keywords, personal preferences, and social connections.
- Supports evaluation of advanced script-based user criteria permissions.
- Indexes comments and attachments on KB articles.
- Supports indexing content from custom or default knowledge article templates, such as FAQs, How-to, What Is, or KCS article templates.
- Supports customization of the ServiceNow URL in Copilot responses as needed for your organization.
- Considers both knowledge base-level and article-level permissions (user criteria) when evaluating article permissions.
- Supports evaluating permissions based on user criteria or role-based permissions.

### Limitations

- The incremental crawl only updates the changed content or any addition or removal of user criteria to any article, not the changes in identity, such as changes in users or user criteria attributes. The identity sync happens only with a full crawl.
- The connector doesn't support indexing of knowledge blocks and evaluation of their user criteria.

## Data types indexed from ServiceNow Knowledge

The connector indexes structured data from ServiceNow knowledge base articles. Indexed content is surfaced in Copilot responses and Microsoft Search results, enabling contextual access to enterprise data.

## Permissions model and access control

Access to indexed ServiceNow data is governed by access control lists (ACLs) and user criteria configured in ServiceNow. Admins can manage permissions to ensure that only authorized users can view specific data types in Copilot and search experiences.
