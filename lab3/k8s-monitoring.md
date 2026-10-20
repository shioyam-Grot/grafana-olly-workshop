<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/k8s-monitoring -->

# 3.2. Query Kubernetes Monitoring data

*In Lab 1.3 you clicked through Workloads, into a pod, into Containers, into the right-sizing gauge. Now ask Assistant for the same answers in plain English and see how the routing changes.*

Open a **new conversation** with Assistant for this section so it starts with a clean slate.

---

## Question 1: Container health

**What's the health of my containers?**

A great opening prompt because it's the kind of question that takes a *human* a few minutes to answer manually - you'd scan the cluster, look for restart counts, check pod events, eyeball errors. Assistant should give you the same summary in one shot, naming the same usual suspects we found in Lab 1.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

Assistant highlights containers with repetitive restarts, bad k8s events, and errors - `productcatalogservice`, `recommendationservice`, and `checkoutservice` show concerning health.

![Container health response](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.1-grafana-assistant-1-d6147d7e5eaa9810c9517451863c5492.png)

</details>

---

## Question 2: Image and version

**What image and version is being used for the `productcatalogservice`?**

Same image-spec lookup as Lab 1.3 - except instead of finding the Containers panel and reading the IMAGE SPEC column, you just ask. Faster, but still verify by expanding the tool call.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

`productcatalogservice:v3.24.5`

![Prompt](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.2-grafana-assistant-1-1244a4bac306a89f6112eba060f38e19.png) ![Answer](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.2-grafana-assistant-2-0728693f68d5287105f25f73474a830f.png)

</details>

---

## Question 3: Right-sizing

**Is the `productcatalogservice` right-sized?**

This was the multi-step deep-dive in Lab 1.3 - click into the container, read the gauge, compare to the recommendation. Assistant pulls the same [right-sizing data](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/optimize-resource-usage/) and delivers the verdict in one prompt. It isn't a new data source - it's a faster route into the same Grafana data you'd otherwise navigate to manually, and trust comes from being able to verify that.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

No - Assistant reports the service as oversized on memory (assigned ~100 MiB but using far less), the same verdict you reached manually in Lab 1.3.

![Right-sizing response](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-grafana-assistant-1-14e694ad6caf7c47a9628146f160a0fe.png)

</details>

---

## Question 4: Deployment namespace *(Optional)*

**What namespace is my app deployed in?**

A single-prompt question that would otherwise take a few clicks through the cluster view. The kind of detail it's annoying to remember where to find.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

Your main application is deployed in the **`ecommerce-prod`** namespace.

![Namespace response](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.4-grafana-assistant-1-e2029e3c937eabb97eae2123bec6716a.png)

</details>

---

## Question 5: Cluster resources *(Optional)*

**What resources does my k8s cluster have?**

Same physical-capacity question from the **Clusters** view in Lab 1.3 - now Assistant produces a full cluster summary including node count, CPU, memory, storage, and pod capacity. The kind of overview you'd otherwise piece together from a few panels.

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

```text
Kubernetes cluster resource summary
Here's a comprehensive overview of your sedemoditl-cluster resources:

🏗️ Cluster architecture
Node configuration:

4 nodes with identical specifications
AWS EC2 instances (ARM-based architecture)
Distributed across 3 availability zones: us-east-1a, us-east-1b, us-east-1c
OS: Amazon Linux VERSION
Kubernetes version: VERSION

💾 Total cluster capacity
Compute resources:

Total CPU: 8 cores (2 cores per node)
Total Memory: 15 GB (~4 GB per node)

📊 Pod capacity
Maximum pods: 330 total (110 per node)
Current allocatable: 330 pod slots available
```

![Prompt](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.5-grafana-assistant-1-06d25b7351ddb8a0af105517883a2e0b.png)

</details>

---

## Question 6: Assistant rule *(Optional)*

**Create a rule scoping Assistant to the `ecommerce-prod` namespace**

The more you use Assistant, the more you'll want it scoped to *your* corner of the world. Rules let you set persistent context - "I'm only ever interested in this namespace" - so it stops searching the whole cluster every time.

Try creating a rule that scopes Assistant to applications and infrastructure in the `ecommerce-prod` namespace. Then open a new conversation and ask "Tell me about what's deployed".

> **Try it:** Ask Grafana Assistant before revealing the answer.

<details>
<summary>Show answer</summary>

If this works, asking the follow-up question and expanding any tool call should show the filter `namespace=ecommerce-prod`.

Opening the rules page:

![Rules page](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-1-30325119ae48886023b4537354b60d66.png)

Click **Create rule**:

![Create rule](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-2-893a23982521eb3e588435c23b38e5cd.png)

Adding the custom rule:

![Adding custom rule](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-3-62900415227351fefbd77a49fe23c73d.png)

The rule in action:

![Rule in action](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-4-c2c5d175b77484662f89ff1b3192274c.png)

</details>
