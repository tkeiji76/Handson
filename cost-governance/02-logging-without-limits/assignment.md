---
slug: logging-without-limits
id: 1fepx2wlki2n
type: challenge
title: Logs Management and Cost Optimization
notes:
- type: text
  contents: |
    # Logs Management and Cost Optimization

    Learn how to implement Logging without Limits to optimize your log costs while maintaining observability.

    We'll cover:
    - Setting up log indexes with quotas
    - Implementing effective log filtering strategies
    - Configuring exclusion filters for cost optimization
    - Managing log spikes and responding to alerts
tabs:
- id: qlmxdos6yo6u
  title: Terminal
  type: terminal
  hostname: lab-host
- id: lfzwunhknlu0
  title: IDE
  type: service
  hostname: lab-host
  path: /
  port: 8080
- id: 193t7nzxwrlv
  title: Datadog
  type: website
  url: https://app.datadoghq.com
  new_window: true
- id: kswg3q3shbjb
  title: Help
  type: website
  url: https://datadoghq.dev/training-lab-support?sandboxId=${_SANDBOX_ID}
difficulty: basic
enhanced_loading: null
---

Datadog log management removes traditional limitations by **decoupling log ingestion from indexing**, which makes it possible to cost-effectively collect, process, and archive all your logs. See the overview: [Logging without Limits](https://docs.datadoghq.com/logs/guide/getting-started-lwl/).

With *Logging without Limits*, you can:
- Dynamically choose which logs to index and retain for troubleshooting and analytics (and override these filters as needed)
- Archive enriched logs in your long-term cloud storage solution at no additional cost
- Observe and query a **Live Tail** of all ingested logs across your entire infrastructure (even the ones you choose not to retain)
- Ingested logs will still generate metrics and security signals even when not indexed !

Setting Up Cost-Effective Log Index Quotas
=========================================================

Best practice is to set up multiple indexes with clear filters and different log retention periods per index.

By creating multiple indexes with different retention periods and sampling rates, you will:
- Reduce indexing costs through strategic filtering
- Optimize storage costs with [varying retention periods](https://docs.datadoghq.com/logs/guide/best-practices-for-log-management/#set-up-multiple-indexes-for-log-segmentation) (e.g., 7 days for debug logs vs 15 months for security logs)
- Prevent cost spikes with daily quotas that halt indexing when limits are reached
- Maintain compliance by ensuring critical logs are retained appropriately

Now, let's look at how to create an index and set up a quota.

In [Logs > Explorer](https://app.datadoghq.com/logs), click the "Log Configuration" button in the upper right corner.

![Log Configuration](../assets/log_config.png)

This takes you to the Logs Settings menu, where you can configure [Pipelines](https://docs.datadoghq.com/logs/log_configuration/pipelines/?tab=source#manage-your-pipelines),[Sensitive Data Scanner](https://docs.datadoghq.com/security/sensitive_data_scanner/), [Log Forwarding](https://docs.datadoghq.com/logs/log_configuration/forwarding_custom_destinations/?tab=http), [Rehydrate from Archives](https://docs.datadoghq.com/logs/log_configuration/rehydrating?tab=amazons3), and, most importantly, [Indexes](https://docs.datadoghq.com/logs/log_configuration/indexes).

> [!NOTE]
> With [Flex logs](https://docs.datadoghq.com/logs/log_configuration/flex_logs/#flex-logs-for-multiple-organization-accounts) we also enable to decouple Storage from Compute lowering even more the price of ingestion.

1. Click on the [(ROUTING) Indexes](https://app.datadoghq.com/logs/pipelines/indexes) section.

![new index](../assets/new_index.png)

2. Create a new index
   1. Set name to `sample-generator`
   2. Set the query to `service:generator`
   3. Under **Configure Storage Tier and Retention**, select a storage tier (retention) - For this lab, it will default to 15 days.
      > [!NOTE]
      > Remember, retention affects cost.
   4. Enable [daily quota](https://docs.datadoghq.com/logs/log_configuration/indexes/#set-daily-quota)
   5. Set a Warning Threshold of 80%
   6. Validate your configs look like the below
      ![sample generator](../assets/sample-generator.png)
   7. Click `Save`


**Index order matters.** Ensure the catch‑all index `main` is listed after `sample-generator`, or the filter won't match.
You can drag and drop an index using the six dots on the right side to reorder.

![drag index](../assets/drag_index.png)

You can now open the index and add an exclusion filter to keep only 10% of those logs.

![set sample](../assets/set_sample.png)

Setting Up Alerts on Index Daily Quotas
=========================================================

In the same way we created a monitor for `estimated_usage` **metrics monitor** earlier on, you can set up monitoring to be alerted when your daily quota is near or has been reached by creating an **event Monitors**.

Refer this [best practices](https://docs.datadoghq.com/logs/guide/best-practices-for-log-management/#alert-on-indexes-reaching-their-daily-quota) documentation for setup steps.

This monitor will alert you immediately when any index reaches its daily quota, allowing for quick response to prevent log data loss.

Alert when an indexed log volume passes a specified threshold
=========================================================

We also recommended that you setup monitors to alert if an indexed log volume in any scope of your infrastructure is growing unexpectedly. This monitor will be a **Logs Monitor**

With that you can setup a **Threshold monitors** with the specific volume limits or known acceptable ranges (e.g., "never exceed 1GB/hour").

Follow this [documentation](https://docs.datadoghq.com/logs/guide/best-practices-for-log-management/#alert-when-an-indexed-log-volume-passes-a-specified-threshold) for more details.

Filtering Out Unnecessary Logs for Cost Optimization
=========================================================

Now that we know how to create and index and monitor on ingestion, let's try to keep only logs that are relevant to our business.
This is the fundation for **Logging without limits** *Only Ingest What You Need*

This section covers multiple strategies to achieve optimal log filtering.

One of the most effective ways to reduce log costs is to filter out unnecessary log levels that don't provide actionable insights in production environments.

Below are typical production log levels:
- ERROR: Critical for debugging
- WARN: Important for monitoring
- INFO: Often unnecessary in production
- DEBUG: Usually only needed during development

The most effective approach is to exclude unnecessary log levels at the index level using Datadoghq's exclusion filters.

To do so, follow the steps below:

1. Go to [Logs > Configuration > Indexes](https://app.datadoghq.com/logs/pipelines/indexes)
2. Click on your `sample-generator` index.
3. Click `+Add Exclusion Filter`
4. Configure the following:
   - Name: `Exclude DEBUG logs`
   - Query: `status:debug`
   - Exclusion Percentage: `100%`
5. Click `Add Exclusion Filter`

Now, navigate to [Logs](https://app.datadoghq.com/logs) and filter for `service:generator`, you should no longer see DEBUG logs:
![Info Error Logs](../assets/info_error_logs.png)

Here are examples of advanced exclusion patterns that can be applied in your environment:

```
# Exclude by multiple criteria
Query: status:debug OR (status:info AND service:cache-service)

# Exclude by environment
Query: env:development status:(debug OR info)

# Exclude by log content
Query: @message:"Starting application" OR @message:"Health check passed"

# Exclude health checks
Query: @http.url_details.path:"/health" OR @http.url_details.path:"/ping"

```
The last two examples focus on specific noisy patterns. Pay attention to recurrent patterns in your environment and implement proper sampling with the above method.

### Agent-Level Filtering

You can also set the Datadoghq Agent to filter logs **before** they’re sent to Datadog, reducing network bandwidth and ingestion costs.

Please note that this will prevent logs from being visible in the **Log Live View**.

Refer to the [Advanced Log Collection Configurations](https://docs.datadoghq.com/agent/logs/advanced_log_collection/?tab=configurationfile) documentation to learn more.


Monitoring Your Filtering Effectiveness
=========================================================

After implementing filtering, we recommended you go through the following verification steps:

1. Validate volume reduction:
   - Monitor `datadog.estimated_usage.logs.ingested_events` in metrics explorer.
   - Compare before/after filtering implementation

2. Check [Plan & Usage](https://app.datadoghq.com/billing/usage) for log costs

3. Validate that there is no operational impact:
   - Ensure error detection isn't affected
   - Verify debugging capabilities are preserved
   - Monitor alert coverage effectiveness

Log Archiving and Rehydration
=========================================================

Log archiving provides cost-effective long-term retention by storing logs in lower-cost cloud storage while maintaining the ability to access them when needed.

Archiving addresses several common challenges:
- Compliance requirements that mandate log retention for years
- Audit trails needed for regulatory purposes
- Cost optimization by using cheaper storage for infrequently accessed data

Datadog can forward enriched logs to your cloud storage before indexing, ensuring logs are preserved regardless of exclusion or sampling rules.

### Setting up Archives

In a production environment, you would configure archives by:
1. Setting up cloud storage (AWS S3, Azure Blob, or Google Cloud Storage) with proper IAM permissions
2. Navigating to [Logs > Configuration > Archives](https://app.datadoghq.com/logs/pipelines/archives)
3. Creating archives with compression enabled to reduce storage costs
4. Configuring query filters to organize different log types

During log enrichment, tag your logs appropriately to optimize retrieval and rehydration costs (for example, by data type such as security vs application logs).

> [!NOTE]
> Archive setup requires external cloud storage credentials, which aren't available in this training environment. However, you can explore the archive configuration UI to understand the options available.

**Reference:** [Log Management archive documentation](https://docs.datadoghq.com/logs/log_configuration/archives/?tab=awss3)

### Log Rehydration - Accessing Archived Data

Rehydration allows you to bring archived logs back into Datadog for analysis when needed.

In a production environment with configured archives, you would:
1. Navigate to [Logs > Configuration > Rehydrate from Archives](https://app.datadoghq.com/logs/pipelines/historical-views)
2. Select specific time range and query filters
3. Choose the destination index (ideally a dedicated short-retention index) and review the estimated costs displayed before proceeding

> [!NOTE]
> Rehydration requires existing archives with historical data, which aren't available in this training environment.

When rehydrating, to limit rehydration costs follow these best practices:
- Use specific queries — only rehydrate the logs you actually need
- Limit time ranges to minimize costs
- Rehydrate into a dedicated index with short retention (for example, 1–7 days)
- Prefer off-peak windows for large rehydrations

> [!IMPORTANT]
> Rehydrated logs are indexed and billed as indexed logs during their retention period. Use narrow queries and short retention to control costs.

**Reference:** [Log Management Rehydration documentation](https://docs.datadoghq.com/logs/log_configuration/rehydrating?tab=amazons3)

Flex Logs
========================================================

Another way to save cost is by decoupling **compute and storage**.

Flex Logs provides a flexible and cost-efficient way to store and query logs by decoupling storage retention from query capacity. Teams can independently adjust retention duration and allocated compute resources, making it ideal for long-term log retention, compliance requirements, and ad-hoc historical queries while optimizing costs.

![Flex Logs](https://datadog-docs.imgix.net/images/logs/log_configuration/flex_logging/logs-spectrum.c1357ada08f0bf899cb7f21d4e7c5b55.png)

You can read more on [Flex Logs documentation](https://docs.datadoghq.com/logs/log_configuration/flex_logs)

> [!NOTE]
> A new feature called Flex Frozen (Archive Search) will soon allow you to query logs stored on Datadog-managed long-term storage.
> 
> Best practices for log tier selection:
> - **Standard Indexing** — For frequently queried, short-term logs (3-15 days), such as application logs
> - **Flex Logs** — For medium-term logs (30-90 days) requiring occasional urgent queries, such as security, transaction, and network logs
> - **Long-term retention** (1+ year) for infrequently queried logs, such as audit and configuration logs:
>   - **Flex Frozen** — For simplified management, compliance, and minimal operational burden
>   - **External Archives** — For lowest storage costs

Next Steps
=========================================================

We have learned how to create a log of different types of monitors:
- **Metrics Monitors**
- **Logs Monitors**
- **Event Monitors**

We have also seen how we can apply **Logging without limits** to get the best value for logs ingested while indexing only what we need for our observability goals.

Let's move on to the next section and see how we can get the best value for each traces consumed using **Ingestion Control**
