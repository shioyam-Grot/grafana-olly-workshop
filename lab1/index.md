<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab1/ -->

# Lab 1 - Introduction to Grafana Observability

In this lab, you'll work through troubleshooting scenarios using the Grafana Cloud Observability apps you've just been introduced to, so you can get hands-on experience with each tool.

## Instructions

Go to the URL in the slides where you have access to a Grafana Cloud stack with:

1. Frontend Observability
2. Application Observability
3. Kubernetes Monitoring

You have 20 minutes to work through the questions.

> **TIP**
>
> Try each question on your own first. The **Show answer** block under each question reveals the walkthrough and screenshots.

- **[1.1. Frontend Observability](frontend-o11y.md)**: Imagine you're on call. Customers are reporting that the ecommerce site is acting up - pages loading slowly, the occasional error. Where do you start? At the same place your users do: in the browser.

- **[1.2. Application Observability](app-o11y.md)**: Frontend Observability told us every page on the ecommerce site is throwing 500s. The browser sees the symptom, but the cause is on the server side. Time to walk into the backend and find out which service is the problem.

- **[1.3. Kubernetes Monitoring](k8s-o11y.md)**: Frontend Observability showed us the ecommerce site is throwing 500s. Application Observability narrowed it down to productcatalogservice - which is failing by design in this demo. Time to drop one more layer and ask Kubernetes what's actually happening to that pod.
