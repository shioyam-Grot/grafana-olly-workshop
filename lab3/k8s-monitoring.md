<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/k8s-monitoring -->

# 3.2. Kubernetes Monitoring のデータをクエリする

*Lab 1.3 では、Workloads → Pod → Containers → right-sizing の gauge、とクリックしてたどりました。今度は Assistant に同じ答えを普段の言葉で尋ね、たどる経路がどう変わるかを見てみましょう。*

このセクションでは、Assistant で**新しい会話**を開き、まっさらな状態から始めてください。

---

## 質問 1: コンテナの健全性

**コンテナの健全性はどうなっていますか？**

最初のプロンプトとして最適な質問です。*人間*が手作業で答えようとすると数分かかる種類の質問だからです。クラスターを見渡し、再起動回数を探し、Pod のイベントを確認し、エラーにざっと目を通す必要があります。Assistant なら同じ要約を一度で出してくれるはずで、Lab 1 で見つけたのと同じおなじみの問題サービスが挙がるはずです。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

Assistant は、再起動を繰り返しているコンテナ、問題のある k8s イベント、エラーを指摘します。`productcatalogservice`、`recommendationservice`、`checkoutservice` の健全性に懸念があると示されます。

![コンテナの健全性に関する回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.1-grafana-assistant-1-d6147d7e5eaa9810c9517451863c5492.png)

</details>

---

## 質問 2: イメージとバージョン

**`productcatalogservice` で使われているイメージとバージョンは何ですか？**

Lab 1.3 と同じイメージ仕様の確認ですが、Containers パネルを探して IMAGE SPEC 列を読む代わりに、ただ聞くだけです。より速くなりますが、tool call を展開して検証することは忘れないでください。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

`productcatalogservice:v3.24.5`

![プロンプト](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.2-grafana-assistant-1-1244a4bac306a89f6112eba060f38e19.png) ![回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.2-grafana-assistant-2-0728693f68d5287105f25f73474a830f.png)

</details>

---

## 質問 3: right-sizing

**`productcatalogservice` のサイズは適切ですか？**

Lab 1.3 では、複数のステップを踏んで深掘りした質問です。コンテナをクリックし、gauge を読み、推奨値と比較しました。Assistant は同じ[right-sizing のデータ](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/optimize-resource-usage/)を取得し、1つのプロンプトで結論を出します。新しい data source ではありません。本来なら手作業でたどる同じ Grafana のデータに、より速くたどり着くための経路です。そして、それを検証できることが信頼につながります。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

いいえ。Assistant は、このサービスのメモリが oversized である（約 100 MiB が割り当てられているが、使用量ははるかに少ない）と報告します。Lab 1.3 で手作業でたどり着いたのと同じ結論です。

![right-sizing に関する回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-grafana-assistant-1-14e694ad6caf7c47a9628146f160a0fe.png)

</details>

---

## 質問 4: デプロイ先の Namespace *（任意）*

**アプリはどの Namespace にデプロイされていますか？**

本来ならクラスタービューを何回かクリックしてたどる必要がある質問ですが、1つのプロンプトで済みます。どこで確認できるかを覚えておくのが面倒な種類の情報です。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

メインのアプリケーションは **`ecommerce-prod`** Namespace にデプロイされています。

![Namespace に関する回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.4-grafana-assistant-1-e2029e3c937eabb97eae2123bec6716a.png)

</details>

---

## 質問 5: クラスターのリソース *（任意）*

**k8s クラスターにはどのようなリソースがありますか？**

Lab 1.3 の **Clusters** ビューで確認した物理キャパシティと同じ質問です。今回は Assistant が、ノード数、CPU、メモリ、ストレージ、Pod のキャパシティを含むクラスター全体の要約を作成します。本来ならいくつかのパネルから情報を組み合わせて把握する必要がある概要です。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

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

![プロンプト](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.5-grafana-assistant-1-06d25b7351ddb8a0af105517883a2e0b.png)

</details>

---

## 質問 6: Assistant のルール *（任意）*

**Assistant の対象を `ecommerce-prod` Namespace に限定するルールを作成してください。**

Assistant を使い込むほど、*自分の*担当範囲に絞って使いたくなってきます。ルールを使うと、「自分が関心があるのはこの Namespace だけ」といった永続的なコンテキストを設定できるので、毎回クラスター全体を検索しなくなります。

`ecommerce-prod` Namespace 内のアプリケーションとインフラストラクチャに Assistant の対象を限定するルールを作成してみてください。その後、新しい会話を開いて「何がデプロイされているか教えて」と聞いてみましょう。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

うまくいっていれば、続けて質問したあとにいずれかの tool call を展開すると、`namespace=ecommerce-prod` というフィルターが表示されるはずです。

ルールのページを開きます。

![ルールのページ](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-1-30325119ae48886023b4537354b60d66.png)

**Create rule** をクリックします。

![Create rule](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-2-893a23982521eb3e588435c23b38e5cd.png)

カスタムルールを追加します。

![カスタムルールの追加](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-3-62900415227351fefbd77a49fe23c73d.png)

ルールが適用された状態です。

![ルールの適用例](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-grafana-assistant-4-c2c5d175b77484662f89ff1b3192274c.png)

</details>
