---
slug: managing-skus
id: 3yf8ktrlyu1r
type: challenge
title: Access Control & SKU Management
notes:
- type: text
  contents: |
    # Cost Governance

    There are multiple ways to enable SKUs in Datadog. This is by design to allow Dev, Ops, and Sec teams to activate products and gain the visibility they need.

    While this flexibility is valuable, it can also create unexpected costs. We’ll focus on maximizing ROI and preventing accidental feature activation.

    In this session, we’ll cover how to deactivate features enabled by mistake and prevent future activation, plus core security best practices.
tabs:
- id: a2cupr6nweqb
  title: Terminal
  type: terminal
  hostname: lab-host
- id: yhmnyvzw3sc4
  title: IDE
  type: service
  hostname: lab-host
  path: /
  port: 8080
- id: prxscvitlvnn
  title: Datadog
  type: website
  url: https://app.datadoghq.com
  new_window: true
- id: umnjtuxxyxxn
  title: Help
  type: website
  url: https://datadoghq.dev/training-lab-support?sandboxId=${_SANDBOX_ID}
difficulty: basic
enhanced_loading: null
---

Role-Based Access Control (RBAC) for Cost Management
=========================================================
Create custom user roles to restrict access to cost-impacting features such as:
- Product activation (APM, Security, RUM, etc.)
- Ingestion configuration changes
- Index retention modifications

Datadog provides three [default roles](https://docs.datadoghq.com/account_management/rbac/?tab=datadoghq-application#datadog-default-roles): **Admin**, **Standard**, and **Read-only**. However, for effective cost governance, it's recommended to define additional custom roles with more granular permissions based on your business rules.

Below are common custom roles that can be added alongside the default ones:
- **Log Analyst**: Can view logs, but cannot modify index retention policies
- **Developer**: Can access APM data, but cannot enable new APM features
- **Observer**: Can view dashboards and metrics, but cannot create billable monitors

### Step-by-Step Role Creation:

1. **Access Role Management**:
   - Log in to [Datadog](https://app.datadoghq.com) as an Admin
   - Navigate to [Organization Settings > Roles](https://app.datadoghq.com/organization-settings/roles)

2. **Enable Custom Roles** (if not already enabled):
   - Click the settings cog icon
   - Select **Enable** for custom roles

3. **Create New Role**:
   - Click **New Role**
   - Enter a descriptive name (e.g., "Cost-Controlled Developer")

4. **Configure Permissions**:
   - **Grant**: Read access to necessary features (APM, Logs, Metrics)
   - **Restrict**: Billing settings, product activation, retention changes
   - Review [Log Management Permissions](https://docs.datadoghq.com/account_management/rbac/permissions/?tab=ui#log-management) for detailed options

5. **Save and Assign**:
   - Click **Save**
   - Assign users to the new role

Beyond logs, you can restrict access to other cost-impacting features:
- **API Management**: Restrict API and application key creation — prevents unauthorized integrations
- **Organization Settings**: Restrict billing access and user management — prevents unauthorized plan changes
- **Automation Tools**: Restrict App Builder and Workflow creation — controls compute resource usage
- **Infrastructure**: Restrict integration installations — prevents unexpected host/container billing

**Complete Reference**: [Datadog RBAC Permissions Documentation](https://docs.datadoghq.com/account_management/rbac/permissions/)

Security Contact Configuration
=========================================================

Datadog uses your security contact to notify you of compromised API keys found on the internet or other security incidents that could impact costs and more.

![sec contact details](../assets/sec_contact_details.png)

You can check the **Safety Center** in [Account name > Organization Settings](https://app.datadoghq.com/organization-settings) to verify that roles and the security contact are properly configured.

![sec contact](../assets/sec_contact.png)

The **Safety Center** also provides quick access to important security events and highlights users pending approval or with high‑level admin permissions.

(PREVIEW) First-Time Usage Notification (New SKU detection)
==========================================================

Datadog provides a Billing Notification Service to clients when they first start to using a product that.

This notification helps clients become aware of new product usage and detect potential out-of-contract usage. Notifications are sent for first-time usage at the sub-org level. This means that even if another org in the same account used the product before, a sub-org will still generate a first-usage notification.

> [!NOTE]
> Each sub-org receives this notification for a given product only once.

![First time usage](../assets/first_time_usage.png)

How to Disable a Datadog Product (API & App Protection example)
=========================================================

API & App Protection is billed per host per month, so accidental org‑wide enablement can be costly.

When AAP was enabled via Datadog UI, follow [these](https://docs.datadoghq.com/security/application_security/troubleshooting/?tab=java#remote-configuration) steps.

When AAP was enabled at the instrumentation level set environment variable: `DD_APPSEC_ENABLED=false` and restart the application or deployment.

Other products offer similar deactivation steps—refer to their documentation for details.

API & App Key Best Practices
=========================================================

Proper API key management is crucial for both security and cost control, as compromised keys can lead to unauthorized usage and unexpected charges.

Below are recommended best practices:
- Rotate API and Application keys at least every 90 days.
- Use dedicated API keys per environment (dev, staging, production).
- Apply the **Principle of Least Privilege** to each key's scope and permissions.
- Issue a unique API key per agent (host) or workload to enable granular revocation and auditing.
- Store keys in a secrets manager; never commit them to source control. Use environment variables or secret mounts at runtime.
- Tag keys with owner, environment, and purpose to simplify audits and revocation.
- Monitor API usage and set alerts for unusual patterns; revoke and rotate immediately if exposure is suspected.

These practices enable granular control, easier access revocation, better audit trails, and help prevent runaway costs from compromised keys.

(PREVIEW) Governance Console
====================================

We have now in preview a Governance Console that allows to:
- Track adoption health at the org level
- Make sure best practices are in place
- Enforce accountability and healthy adoption

![Governance Console](../assets/governance_console.png)

Lab Summary
=========================================================
Congratulations! You've completed the Cost Governance workshop!
