<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab2/ -->

# Lab 2 - Knowledge Graph による根本原因分析

このラボでは、**Grafana Cloud Knowledge Graph**、特に **Entity Graph** と **RCA Workbench** を使ってトラブルシューティングのシナリオに取り組みます。

Lab 1 のアプリでは簡単にはできないことや、同じ作業が Knowledge Graph を使うとどれだけ簡単になるかを体験していただくのが狙いです。

## 手順

スライドに記載された URL にアクセスしてください。Knowledge Graph が有効になった Grafana Cloud スタックにアクセスできます。

質問に取り組む時間は20分です。

> **ヒント**
>
> まずは自分で各質問に挑戦してみてください。各質問の下にある **答えを表示** ブロックを開くと、手順の解説とスクリーンショットが表示されます。

- **[2.1. Knowledge Graph を使う](knowledge-graph.md)**: Lab 1 では、Frontend O11y → App O11y → Kubernetes Monitoring と、アプリを行き来しながら手がかりを手作業でたどりました。うまくいきましたが、クリック数はかなり多くなりました。今度は、あらゆるものの関係性があらかじめマッピングされている状態で、同じ調査がどのように進むかを見てみましょう。

- **[2.2. 従来の Observability アプリと比較する](compare-with-classic-apps.md)**: Knowledge Graph を使って、いくつかの質問に数クリックで答えられました。今度は同じ質問に Lab 1 のツール（App O11y、K8s Monitoring）だけで答えてみて、かかった時間を計ってみましょう。
