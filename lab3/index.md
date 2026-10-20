<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/ -->

# Lab 3 - Observability with AI

In this lab, you'll work through troubleshooting scenarios from Lab 1, but this time using AI - specifically the [Grafana Assistant](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/) - to find the answers.

## Instructions

You'll use the **same environment as Lab 1**. Go to the URL in the slides that references the Lab 1 environment, which has:

1. Frontend Observability
2. Application Observability
3. Kubernetes Monitoring

Click the **Grafana Assistant** button in the upper right corner to get started. Keep the same prompt window across questions within a section for context continuity, but start a new prompt window when you change sections so you can see how Assistant processes each request fresh. In the real world, you'd keep one prompt for a single investigation.

![Grafana Assistant entry point](https://grafana.github.io/observability-workshop-breakouts/assets/images/0.1-grafana-assistant-5692ca4ca767d2d550cd20da3d87afc4.png)

You have 20 minutes - prioritize the core questions, then try the extra credit if time allows.

> **NOTE**
>
> Because of the variable nature of an LLM assistant, results will not always match between users or match what we found in earlier exercises. If Assistant isn't doing what you want, prompt it again to nudge it in the right direction.

### Things to look out for

**Tool usage in Assistant** - LLMs can hallucinate, so Assistant shows its working. Expand the tool calls to see what it's thinking so you can trust its conclusions.

**An error in the Assistant** - Assistant makes mistakes like humans. We feed the error back so it can reconsider and iterate. If it truly can't conclude, it'll usually tell you (after trying for a while).

**A graph or logs visualization** - Assistant uses Grafana's visualization library to *show* rather than just *tell*, building trust and breaking up walls of text.

**Suggested next-step buttons** - Most of the time Assistant suggests follow-up questions or actions. You can also just keep prompting.

- **[3.1. Query Frontend Observability data](frontend-o11y.md)**: You've now solved the productcatalog mystery two ways - one way through Lab 1's apps, and a second, shorter way through Knowledge Graph in Lab 2. There's a third option: don't navigate anywhere. Just ask.

- **[3.2. Query Kubernetes Monitoring data](k8s-monitoring.md)**: In Lab 1.3 you clicked through Workloads, into a pod, into Containers, into the right-sizing gauge. Now ask Assistant for the same answers in plain English and see how the routing changes.

- **[3.3. Query Span Metrics](span-metrics.md)**: In Lab 1.2 you walked through Application Observability service-by-service to track the productcatalog crash loop. Here you'll do it again, but Assistant drives - and you'll see whether it lands the same conclusions you reached manually.
