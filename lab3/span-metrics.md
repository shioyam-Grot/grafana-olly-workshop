<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/span-metrics -->

# 3.3. Span Metrics をクエリする

*Lab 1.2 では、Application Observability でサービスを1つずつ確認しながら、productcatalog の crash loop を追跡しました。ここではもう一度同じことを行いますが、今回は Assistant が主導します。手作業でたどり着いたのと同じ結論に Assistant がたどり着けるかを確認しましょう。*

Span Metrics は OpenTelemetry のトレースから生成されるため、パネルを1つも開かなくても、Assistant はサービスの健全性、依存関係、productcatalog の一連の状況について推論できます。

このセクションでは、Assistant で**新しい会話**を開いてください。

---

## 質問 1: サービスの健全性

**サービスは健全ですか？**

Service Inventory を開いてエラー列やレイテンシ列にざっと目を通すことを、自然言語で行うのと同じです。Assistant がどのサービスを指摘するかに注目してください。Lab 1.2 で見つけたのと同じサービスのはずです。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

いいえ。Assistant は、Lab 1.2 で見つけたのと同じ不健全なサービス（`productcatalogservice`、`recommendationservice`、`checkoutservice` など）を指摘します。

![サービスの健全性に関する回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-grafana-assistant-1-c27e592f643f26efddf58414bd91e85c.png)

</details>

---

## 質問 2: サービス間の連携

**`productcatalogservice` と連携しているのは何ですか？downstream に問題はありますか？**

Lab 1.2 では、Inbound パネルと Outbound パネルを読んで、productcatalogservice とやり取りしているサービスを調べました。今度は同じ質問を一度にまとめて聞いてみましょう。Assistant は「何がこれを呼び出しているか」と「今何が壊れているか」を相関付けるので、依存関係マップ*と*影響範囲を1つの回答で得られます。単にデータを取得しているのではなく、データを結び付けています。依存関係と現在の健全性を1つのプロンプトで引き出すのは、従来のアプリでは2つの別々の画面で行う必要がある作業です。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

**`productcatalogservice` と連携しているサービス:**

1. `checkoutservice`
2. `frontend`
3. `recommendationservice`
4. `inventoryservice`

**確認された downstream の問題:**

1. checkoutservice が不安定になっている。
2. サービス間のやり取りのパターン:
   - gRPC 呼び出しの失敗: `GetProduct` メソッドと `ListProducts` メソッド。
   - 複数リージョンでの障害: 3つのリージョンすべてが同時に影響を受けている。
   - 時間的な相関: checkoutservice のエラーは、productcatalogservice の再起動期間中に発生している。

![プロンプト](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-1-1b51dc48e75d2a4e6f762dcb35b85907.png) ![調査結果 1](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-2-02ac8cf0552ae72f32ee33cc1de68a6a.png) ![調査結果 2](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-3-dc895896782f80aa5783421dea1c5269.png) ![調査結果 3](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-4-f521a0b7ac293c20108b782b9edd2258.png) ![調査結果 4](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-5-c23ab623b3074d84081371fb2f6476ef.png) ![回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-grafana-assistant-6-efbc356163b82d01d95b665cf5e75d0b.png)

</details>

---

## 質問 3: Postgres のバージョン *（任意）*

**アプリにデプロイされている postgres のバージョンは何ですか？**

インフラストラクチャの属性をさっと調べる質問です。普段ならどこを見ればよいかを覚えておくか、Knowledge Graph を開いてプロパティを読む必要がある種類の質問です。Assistant なら、どちらの方法も知らなくても答えを出してくれるはずです。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

**PostgreSQL のバージョン: 16.0**

![プロンプト](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-1-51d5c7d8f9735a93acf1433b0973a984.png) ![調査結果 1](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-2-7c1f9f4bdd6b20d8478659760d5e5f45.png) ![調査結果 2](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-3-9513d72e2a6e5edfff3c2bc04a9faf4c.png) ![回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-grafana-assistant-4-87443c7cfb00f63d5e173e0b4762f0fa.png)

</details>

---

## 質問 4: サービスのトリアージ *（任意）*

**`productcatalogservice` で何が起きていますか？**

これがワークショップの最後の質問です。Lab 1 から Lab 2 までの調査全体を、1つのプロンプトに凝縮します。Assistant は再起動、エラーログ、トレース、downstream への影響を相関付けて、トリアージの要約を作成するはずです。本来ならいくつもの画面を行き来し、かなりの回数クリックしてまとめる必要がある評価です。

Assistant の回答を、遠回りして見つけた内容と比べてみてください。どのくらい近いでしょうか？

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

```text
📈 Current Status:
Service: Partially functional (restarts every few minutes)
User impact: HIGH - catalog lookups likely failing intermittently
Urgency: CRITICAL - needs immediate developer intervention

The productcatalogservice is in a crash-restart loop due to a combination of database connection exhaustion and runtime panics. It needs immediate code fixes and database configuration changes.
```

![プロンプト](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-1-40da9f0235e9aa3896f61935a83b7e46.png) ![調査結果 1](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-2-3a9d24b86b863b5cfa3dcb40d2f4d934.png) ![調査結果 2](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-3-2885d584ada32aa35cf35d21efa748a3.png) ![調査結果 3](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-4-3b043ef03c892e5307e0aa5f81c2b61b.png) ![調査結果 4](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-5-536add7b41ed09adfb6a35d064fefe95.png) ![回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-grafana-assistant-6-811abd5cb7c63199b7515c767dd0c300.png)

</details>
