---
slug: custom-metrics
id: io0dtbfr2bx4
type: challenge
title: Metrics without Limits and Cost Optimization
notes:
- type: text
  contents: |
    # Custom Metrics Cost Optimization & Comprehensive Usage Monitoring

    Custom metrics can become a major cost driver when cardinality spirals out of control. An unbounded tag (for example, user IDs) can multiply metric cardinality significantly.

    Datadog provides tooling to help you understand and govern custom metrics, including visibility into usage and tag-driven cardinality.

    In this lab, you'll practice cardinality optimization techniques and usage monitoring across Datadog products.
tabs:
- id: tw0rlvmukfhg
  title: Terminal
  type: terminal
  hostname: lab-host
- id: xsoalbwzskta
  title: IDE
  type: service
  hostname: lab-host
  path: /
  port: 8080
- id: uq0ysub1m7oy
  title: Datadog
  type: website
  url: https://app.datadoghq.com
  new_window: true
- id: 3inc1vvlo5yg
  title: Help
  type: website
  url: https://datadoghq.dev/training-lab-support?sandboxId=${_SANDBOX_ID}
difficulty: basic
enhanced_loading: null
--- 

Many standard integrations provide built‑in metrics that are not billed as custom. When possible, utilize native integrations (for example, the NGINX integration) over emitting your own custom metrics.

If you need long‑term trends derived from logs or traces, consider generating metrics (for example, `gauge` or `count`) from that data. Metrics have long‑term retention suitable for trend analysis.

Each host plan includes an allotment of custom metrics by default.

![pricing_metrics](../assets/pricing_metrics.png)

The most important part is to understand how custom metrics are priced and avoid using **unbounded** tags on metrics.
Let's understand why.

The Custom Metrics Cost Challenge
=========================================================

Custom metrics can quickly become expensive when you accidentally include **unbounded tags** - tags with unlimited unique values like user IDs, request IDs, or timestamps.

Each unique tag combination counts as a separate custom metric for billing.

### Understanding Cardinality Mathematics

Custom metrics pricing is based on **cardinality** - the total number of unique tag combinations:

```
Formula: Cardinality = Tag1_Values × Tag2_Values × Tag3_Values × ...
```

### Smart vs. Costly Metric Choices:

**Good Example - Bounded Tags:**
- Metric: `ecommerce.orders.total`
- Tags: `environment` (3 values) + `region` (4 values)
- **Cardinality: 3 × 4 = 12 custom metrics**
- **Cost impact: Minimal**

**Problematic Example - Unbounded Tags:**
- Same metric: `ecommerce.orders.total`
- Tags: `environment` (3 values) + `customer_id` (50,000 unique customers)
- **Cardinality: 3 × 50,000 = 150,000 custom metrics**
- **Cost impact: Massive - depends on the contract**

**The key difference:** Adding `customer_id` creates an **unbounded** dimension that multiplies costs exponentially. Tags like `user_id`, `session_id`, or `request_id` should never be used on metrics.

Cardinality Management
=========================================================

Datadog provides comprehensive tools to identify, optimize, and monitor custom metrics costs.

To identify high-cardinality custom metrics, navigate to [Metrics > Volume](https://app.datadoghq.com/metric/volume?sort=-volume_total).

Click the `frontend.user.count` metric to view its tag breakdown and related assets (for example, monitors and dashboards).

Here you can quickly find that the `user` tag is causing high cardinality for the metric.
![metrics_details](../assets/metrics_details.png)

Under **Manage Tags**, remove the unbounded `user` tag from the metric’s group‑bys to reduce cardinality.
![Remove Tag](../assets/remove_tag.png)

Don't forget to save changes by clicking **Update Metric**.

Proactive Monitoring
=========================================================
Under [Metrics > Explorer](https://app.datadoghq.com/metric/explorer), search for `datadog.estimated_usage.metrics.custom` to find usage metrics for monitoring custom metrics spend.

In about 5 minutes, you should see the metric plateau after the time you made the change to exclude the `user` tag from the `frontend.user.count`:
![Metric Plateau](../assets/metric_plateau.png)

Use these metrics to create an [anomaly detection monitor](https://app.datadoghq.com/monitors/create/anomaly) to catch unexpected spikes before they impact your budget.

Next Steps
=========================================================
Now that you understand how custom metrics are used and managed in Datadog—and how to avoid high cardinality—let's move to the next section.

In the next lab, we'll explore SKU management and advanced cost allocation strategies to complete your cost governance mastery.

