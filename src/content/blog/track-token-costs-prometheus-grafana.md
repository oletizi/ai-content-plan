---
title: "Track and Visualize Token Costs with Prometheus and Grafana"
date: 2025-05-28
description: "Learn how to monitor LLM token usage and inference costs using Envoy AI Gateway with Prometheus and Grafana dashboards."
---

# Track and Visualize Token Costs with Prometheus and Grafana

As your usage of large language models (LLMs) grows, so do your costs. But tracking API billing from providers like OpenAI or Anthropic isn’t straightforward—especially when multiple teams, apps, or services are sharing access.

With **Envoy AI Gateway**, you can expose fine-grained **metrics for token usage, request rates, and response patterns**, and then visualize them using Prometheus and Grafana.

This post walks you through setting up a full stack for LLM cost observability.

---

## Why You Need Token-Level Visibility

Even small prompts can add up quickly. Without visibility into usage patterns, platform teams often face:

- Inability to forecast monthly API spend
- Difficulty attributing costs to teams or services
- No alerting before budget thresholds are hit
- No insight into abnormal spikes or regressions

By integrating Envoy AI Gateway with Prometheus and Grafana, you can answer questions like:

- Who’s using the most tokens?
- Which apps are driving the highest cost?
- When did a sudden spike begin?
- Are latency and errors affecting certain tenants?

---

## Architecture Overview

```
[ App / CLI / Service ]
        |
   [ Envoy AI Gateway ]
        |
   ↙︎         ↘︎
Prometheus   →  OpenAI / Claude / LLM
     |
  Grafana Dashboards
```

---

## Step 1: Expose Metrics from Envoy AI Gateway

Envoy exposes metrics in a Prometheus-compatible format. When deploying Envoy AI Gateway:

- Enable Prometheus scraping on the gateway’s metrics port
- Export custom metrics for:
  - `llm_tokens_in`
  - `llm_tokens_out`
  - `llm_total_cost_usd`
  - `llm_requests_total`
  - `llm_errors_total`
  - `llm_latency_seconds`

Each metric should be labeled by:
- Tenant / team
- Endpoint
- Model
- HTTP response code

---

## Step 2: Configure Prometheus

In your Prometheus config:

```yaml
scrape_configs:
  - job_name: 'envoy-ai-gateway'
    static_configs:
      - targets: ['envoy-gateway-service:9102']
```

Ensure the port matches your gateway’s `stats` listener. You can also use Kubernetes service discovery for dynamic environments.

---

## Step 3: Create Grafana Dashboards

Use Grafana to create dashboards that include:

- 🔢 **Tokens used per day / week / month**
- 📊 **Top consumers by team or service**
- 💵 **Estimated cost (tokens × $/token)**
- ⏱️ **Latency percentiles and errors**
- 🚨 **Alerts for quota breaches or spikes**

Sample query to estimate token cost:

```promql
sum by (team) (llm_tokens_in + llm_tokens_out) * 0.00002
```

Adjust the multiplier for your OpenAI or Claude pricing.

---

## Bonus: Alerts and Budgets

Set up alert rules in Prometheus to detect:

- Unusual spikes in token usage
- Daily budget exceeded per tenant
- Missing metrics (e.g. exporter crash)

---

## Outcome

With this stack in place, you’ll gain:

- Cost attribution for LLM usage
- Predictive usage insights across environments
- Alerts before overruns or service disruptions
- Transparency for internal chargeback models

---

📈 **Get visibility into your AI costs today**  
📘 [See full metrics and dashboard examples](#)  
🚀 [Deploy Envoy AI Gateway + Prometheus stack](#)
