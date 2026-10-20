<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/span-metrics -->

# 3.3. Query Span Metrics

*In Lab 1.2 you walked through Application Observability service-by-service to track the productcatalog crash loop. Here you'll do it again, but Assistant drives - and you'll see whether it lands the same conclusions you reached manually.*

Span metrics are derived from your OpenTelemetry traces, so Assistant can reason about service health, dependencies, and the productcatalog story without you opening a single panel.

Open a **new conversation** with Assistant for this section.

---

## Question 1: Service health

**Are my services healthy?**

The natural-language equivalent of opening the Service Inventory and scanning the error/latency columns. Watch which services Assistant flags - they should be the same ones you spotted in Lab 1.2.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

No - Assistant flags the same unhealthy services you spotted in Lab 1.2, including `productcatalogservice`, `recommendationservice`, and `checkoutservice`.

![Service health response](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-grafana-assistant-1-c27e592f643f26efddf58414bd91e85c.png)

</details>

---

## Question 2: Service interactions

**What interacts with the `productcatalogservice`? Any downstream issues?**

In Lab 1.2 you read the Inbound and Outbound panels to figure out who talks to productcatalog. Now ask the same question, but in one shot - Assistant correlates "what calls this" with "what's broken right now" so you get the dependency map *and* the impact in a single answer. It isn't just retrieving data, it's joining it: pulling dependencies and current health in one prompt is something the classic apps would make you do in two separate views.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

**Services that interact with `productcatalogservice`:**

1. `checkoutservice`
2. `frontend`
3. `recommendationservice`
4. `inventoryservice`

**Confirmed downstream issues:**

1. Checkout service instability.
2. Service interaction patterns:
   - gRPC calls failing: `GetProduct` and `ListProducts` methods.
   - Multiple regional failures: all 3 regions affected simultaneously.
   - Time correlation: checkout errors occur during catalog service restart periods.

![Prompt](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-1-1b51dc48e75d2a4e6f762dcb35b85907.png) ![Findings 1](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-2-02ac8cf0552ae72f32ee33cc1de68a6a.png) ![Findings 2](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-3-dc895896782f80aa5783421dea1c5269.png) ![Findings 3](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-4-f521a0b7ac293c20108b782b9edd2258.png) ![Findings 4](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-5-c23ab623b3074d84081371fb2f6476ef.png) ![Answer](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-6-efbc356163b82d01d95b665cf5e75d0b.png)

</details>

---

## Question 3: Postgres version *(Optional)*

**What version of postgres is deployed in my app?**

A quick infrastructure-attribute lookup. The kind of question that'd normally need you to remember where to look - or pop into Knowledge Graph to read the properties bag. Assistant should resolve it without you needing to know either path.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

**PostgreSQL version: 16.0**

![Prompt](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-1-51d5c7d8f9735a93acf1433b0973a984.png) ![Findings 1](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-2-7c1f9f4bdd6b20d8478659760d5e5f45.png) ![Findings 2](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-3-9513d72e2a6e5edfff3c2bc04a9faf4c.png) ![Answer](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-4-87443c7cfb00f63d5e173e0b4762f0fa.png)

</details>

---

## Question 4: Service triage *(Optional)*

**What's going on with the `productcatalogservice`?**

This is the workshop's final question: the entire investigation from Lab 1 → Lab 2, reduced to a single prompt. Assistant should correlate restarts, error logs, traces, and downstream impact into a triage summary - the kind of assessment that would otherwise take you several views and a fair amount of clicking to assemble.

Compare its answer to what you found the long way. How close is it?

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

```text
📈 Current Status:
Service: Partially functional (restarts every few minutes)
User impact: HIGH - catalog lookups likely failing intermittently
Urgency: CRITICAL - needs immediate developer intervention

The productcatalogservice is in a crash-restart loop due to a combination of database connection exhaustion and runtime panics. It needs immediate code fixes and database configuration changes.
```

![Prompt](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-1-40da9f0235e9aa3896f61935a83b7e46.png) ![Findings 1](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-2-3a9d24b86b863b5cfa3dcb40d2f4d934.png) ![Findings 2](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-3-2885d584ada32aa35cf35d21efa748a3.png) ![Findings 3](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-4-3b043ef03c892e5307e0aa5f81c2b61b.png) ![Findings 4](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-5-536add7b41ed09adfb6a35d064fefe95.png) ![Answer](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-6-811abd5cb7c63199b7515c767dd0c300.png)

</details>
