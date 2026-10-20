<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab1/k8s-o11y -->

# 1.3. Kubernetes Monitoring

*Frontend Observability showed us the ecommerce site is throwing 500s. Application Observability narrowed it down to `productcatalogservice` - which is failing by design in this demo. Time to drop one more layer and ask Kubernetes what's actually happening to that pod.*

Grafana Cloud [Kubernetes Monitoring](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/) is the unified view of your cluster - workloads, pods, containers, and nodes - with pre-built dashboards, alerts, and right-sizing recommendations all in one place. No app switching, no PromQL required.

Navigate to the **Kubernetes Monitoring** app in Grafana.

![Kubernetes Monitoring - getting started](https://grafana.github.io/observability-workshop-breakouts/assets/images/0.3-k8s-instructions-424b45e946c8129e91d56323b7e1a02b.png)

---

## Question 1: Pod restarts

**How many restarts has the `productcatalogservice` had?**

When a container panics or fails its liveness probe, Kubernetes kills it and starts a new one. That restart counter is the most direct signal that something at the application layer is killing the process - the 500s the frontend is throwing back in the Frontend Observability section are downstream of this number going up. We already know productcatalog is unhealthy - the question is whether K8s is seeing the same story we saw in App O11y.

There are two routes to the answer: through Application Observability (where you already are), or directly through the Kubernetes Monitoring app.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

The exact number varies - the service is constantly failing (by design). Use either of the two paths below to read the current value.

### Option 1 - Via Application Observability

1. Open the Application Observability app.
2. Make sure you're on the **Services** tab.
3. Click on `productcatalogservice`.
4. Click the **Kubernetes** button and find the `k8s.pod.name` entry.
5. See the **number of restarts** entry under **Pod information**.

![Pod information showing restarts](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-1-cd9ca6bbc93906c88f8e3b7be66f5d81.png)

### Option 2 - Via Kubernetes Monitoring

1.  Open the Kubernetes Monitoring app under the **Observability** menu.
2.  Click on **Workloads**.

   ![Workloads](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-2-213e34b93a9859d92da4894e9a5b7a9f.png)
3.  Either scroll the list or filter at the top.

   ![Filtered workloads](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-3-1c89baaea9bf12224339c50de81f0c51.png)
4.  Click the `productcatalogservice` entry that does **not** contain `europe`. This opens the deployment overview. Click a pod name in the **pods** table to get pod-level data.

   ![Workload detail](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-4-2192056ec78fc5b475088efe7a7b9c27.png)
5.  See the **number of restarts** entry under **Pod information**.

   ![Pod information](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-1-cd9ca6bbc93906c88f8e3b7be66f5d81.png)

</details>

---

## Question 2: Image and version

**What image name and version is being used for the `productcatalogservice`?**

The pod is crash looping. Now the next question every responder asks: *what version is actually running?* The container image tag is the difference between "this is the regression we just shipped" and "this has been broken for weeks and nobody noticed." It's recorded right on the pod.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

```text
us-central1-docker.pkg.dev/field-engineering-grafana/application-environment/productcatalogservice:v{VERSION}
```

`{VERSION}` may change as the demo app is updated.

**How to find it:**

1. Get to the pod view of the productcatalog service (see [the restarts question](#question-1-pod-restarts)).
2. Read the **IMAGE SPEC** column in the **Containers** panel.

![Image spec](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-k8s-o11y-1-1f0458ee2f2615156da4166a22598eee.png)

</details>

---

## Question 3: Memory assigned

**How much memory is assigned to the `productcatalogservice`?**

Every workload declares a memory request and limit when it's deployed - those are the guardrails Kubernetes enforces. Before we can have any opinion about whether the pod is sized correctly, we need to know what it asked for.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

**100 MiB**

**How to find it:**

1. Open the workload or pod view of the productcatalog service (see [the restarts question](#question-1-pod-restarts)).
2. Hover over the dashed line in the **Workload Memory** panel.

![Workload memory](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-k8s-o11y-3bf28a32aab41af147c077352304c423.png)

</details>

---

## Question 4: Right-sizing

**Is the `productcatalogservice` over- or under-sized on memory?**

The workload was assigned 100 MiB - but is it actually using anywhere close to that? Grafana Kubernetes Monitoring [tracks requested vs. actual usage](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/optimize-resource-usage/) and surfaces the gap as a colored gauge. Glance at it, get your answer.

> **Why this matters:** Overprovisioned workloads tie up capacity other workloads could be using. Per Grafana's docs, the right-sizing data "enables teams to adjust requests downward and reclaim unused capacity on existing hardware" - capacity (and cost) you don't have to buy.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

**Oversized.**

**How to find it:**

1.  Open the pod view of the productcatalog service (see [the restarts question](#question-1-pod-restarts)).
2.  Click the `productcatalogservice` container in the **Containers** table.

   ![Containers table](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-1-84908dd330242ef6a4bd57fac5801153.png)
3.  The current size is 100 MiB but the gauge shows 81 MiB in the red - meaning it's oversized.

   ![Container memory gauge](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-2-71e7b5b0ea4b8fd5dca7b88ec29d48f0.png) ![Container memory detail](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-3-4a45120d60f45fbf2f747261de2a169b.png)

</details>

---

## Question 5: Memory recommendation

**What's the memory recommendation for the `productcatalogservice`?**

The same view that flagged the workload as oversized tells you exactly what to set it to. No guessing, no math - just a number derived from observed usage, ready to drop into your Helm chart or kustomization.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

Between **15 MiB and 20 MiB**.

**How to find it:**

1.  Following on from [the previous question](#question-4-right-sizing):

   ![Memory gauge](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-2-71e7b5b0ea4b8fd5dca7b88ec29d48f0.png) ![Memory recommendation](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.5-k8s-o11y-aacc098bd1b21f373e4162a58447dc25.png)

</details>

---

## Question 6: Cluster capacity *(Optional)*

**How much memory and CPU are allocated in the K8s cluster?**

Zoom out from the single workload. The **Clusters** view shows the physical capacity of every cluster in your fleet - the upper bound everything else has to fit inside. Useful when you're wondering whether you actually have room to scale a workload up, or whether you're already maxed out.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

**CPU:** 8 cores
 **Memory:** 15 GB

**How to find it:**

1.  Open the Kubernetes Monitoring app under the **Observability** menu.
2.  Click on **Clusters**.

   ![Clusters](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.6-k8s-o11y-1-b727b4c98c559ee1f36f704d19e710e2.png)
3.  Click the cluster.
4.  Look at the **Physical capacity of cluster** line in both the **Cluster CPU** and **Cluster Memory** panels.

   ![Cluster capacity](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.6-k8s-o11y-2-7a38c23dfa70e04559a0c50e39d4cf1e.png)

</details>

---

## Question 7: Cluster energy usage *(Optional)*

**What's the total energy usage of the K8s cluster?**

One more stop while we're at the cluster level. Kubernetes Monitoring exposes [fleet energy usage](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/) too - so if anyone on your team is tracking sustainability metrics alongside resource efficiency, the data is already there.

> **Try it:** Find the answer in Grafana before revealing it below.

<details>
<summary>Show answer</summary>

Around **200 Wh**.

**How to find it:**

1.  Open the Kubernetes Monitoring app under the **Observability** menu.
2.  Click on **Clusters**.

   ![Clusters](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.6-k8s-o11y-1-b727b4c98c559ee1f36f704d19e710e2.png)
3.  Click the cluster.
4.  Click the **Energy** tab.
5.  Look at the `Total` series - it may be hidden behind other series, so check the legend.

   ![Energy usage](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.7-k8s-o11y-c84390ef1eddd603f1cd79988d2a8f91.png)

</details>
