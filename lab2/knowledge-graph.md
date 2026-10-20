<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab2/knowledge-graph -->

# 2.1. Knowledge Graph を使う

*Lab 1 では、Frontend O11y → App O11y → Kubernetes Monitoring と、アプリを行き来しながら手がかりを手作業でたどりました。うまくいきましたが、クリック数はかなり多くなりました。今度は、あらゆるものの関係性があらかじめマッピングされている状態で、同じ調査がどのように進むかを見てみましょう。*

Grafana Cloud の [Knowledge Graph](https://grafana.com/docs/grafana-cloud/knowledge-graph/) は、すでに送信しているテレメトリから、サービス、インフラストラクチャ、そしてそれらの間の関係を自動的に検出します。メトリクス、ログ、トレースを1つのワークフローにつなげるので、複雑なクエリを書かずにトラブルシューティングできます。

このラボでは、次の3つの画面を使って Knowledge Graph を操作します。

- **[Entity Catalog](https://grafana.com/docs/grafana-cloud/knowledge-graph/troubleshoot-infra-apps/explore-entity-catalog/)** - 主な入り口です。検出されたすべての entity（サービス、Pod、Namespace）が、その健全性や insight とともに一覧表示されるので、対応が必要なものをひと目で把握できます。
- **[Entity Graph](https://grafana.com/docs/grafana-cloud/knowledge-graph/troubleshoot-infra-apps/explore-entity-graph/)** - それらの entity 同士の関係を、ネットワーク図として視覚的に表示します。
- **[RCA Workbench](https://grafana.com/docs/grafana-cloud/knowledge-graph/troubleshoot-infra-apps/workbench/)** - Knowledge Graph の上に構築された機能です。insight を Timeline 上に並べ、時間軸と依存関係で相関付けられた潜在的な原因を調査できます。

---

## 質問 1: 最も影響を受けているサービス

**過去24時間で、最も状態が悪いと思われるサービスはどれですか？それはなぜですか？**

これは Lab 1 で、ページごとのエラー → サービスのエラー率 → 再起動回数、と遠回りして答えた質問です。Knowledge Graph なら、*画面を開いた瞬間に最初に表示されます*。**Entity Catalog** の左側にあるサービス一覧は **insights**（各 entity で発火している異常、エラー、障害の件数）の多い順に並んでいるため、最も状態の悪いサービスが一番上に来ます。insights で並べ替えるだけで、Lab 1 のトリアージ作業全体を置き換えられます。エラーはあらかじめ entity と相関付けられているので、3つの異なるアプリをまたいで追いかける必要はありません。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**`frontend`** サービスです。他のどのサービスよりも多くの insight が発火しています。frontend は API ゲートウェイなので、すべてのサービスのトラフィックがここを通り、エラーが上流に伝播して insight が発火します。

確認された insight:

- **エラー / 異常:**
   - `INBOUND:LatencyAverageAnomaly`
   - `INBOUND:span_errors::ErrorRatioBreach`
   - `OUTBOUND:LatencyAverageAnomaly`
   - `OUTBOUND:span_errors::ErrorRatioBreach`

**確認方法:**

1.  **Entity Catalog** を開きます。左側のサービス一覧は **insights** の順に並んでいるので、最も状態の悪い entity が上位に表示されます。

   ![insights 順に並んだすべての entity](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.1-knowledge-graph-1-7e65e9085b125b46452a73dce32ce238.png)
2.  entity にカーソルを合わせると、その insight が表示されます。

   ![entity の insight の popover](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.1-knowledge-graph-2-67120ab338e295375c17c8bbed0c7cad.png)

</details>

---

## 質問 2: 接続されているサービス

**過去24時間で、`productcatalogservice` に接続されているアプリケーションサービスはいくつありますか？**

Lab 1.2 では、`productcatalogservice` をクリックしてから、Inbound パネルと Outbound パネルを別々に確認しましたね。Knowledge Graph はテレメトリから依存関係マップ全体を自動で構築するので、「このサービスとやり取りしているのはどのサービスか」に1つの画面で答えられます。グラフでも検索でも、どちらでも構いません。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**7つのサービス**です。Alloy レシーバーはアプリケーションの一部ではないので数えません。

### 方法 1 - Entity Graph から確認する

1.  Entity Graph を開き、`productcatalogservice` を見つけてクリックし、**Connected Entities** を展開します。

   ![Connected Entities](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.2-knowledge-graph-1-2f88ad35deb88ed962633c03c2afd9f4.png)
2.  **Services** をクリックします。
3.  接続済みとして表示されているサービスの数を数えます。

   ![接続されているサービス](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.2-knowledge-graph-2-973dc1e8e877f245f00deec50366dd28.png)

### 方法 2 - 検索バーから確認する

1.  Entity Graph の検索バーに `productcatalogservice connected services` と入力します。
2.  グラフの結果で `productcatalogservice` をクリックします。青いリングが表示されます。
3.  色付きのリングが表示されている接続先 entity の数を数えます。

   ![検索による表示](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.2-knowledge-graph-3-fcd61ec798c534cdffaa398df7ead84f.png)

</details>

---

## 質問 3: cart の Pod

**cart サービスにはいくつの Pod がありますか？**

Connected Entities はサービスに限らず、あらゆる種類の entity に使えます。つまり「このサービスに属する Pod はいくつか？」は「接続されているサービスはいくつか？」と同じ操作で、entity の軸が違うだけです。同じ UI で、別の質問に答えられます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**1つ**です。

### 方法 1

1.  Entity Catalog から始めます。何も表示されていない場合は、**Show All Services** をクリックするか、検索バーに入力します。

   ![Show all services](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-knowledge-graph-1-2324f574bf84c36ec838d9f0cc079f02.png)
2.  検出された*すべての*サービスが表示されます。色付きのリングは例の画像と異なる場合があります。

   ![すべての entity](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-knowledge-graph-2-b1ca8412ec5a83c6a98e36fed3e34d36.png)
3.  右上のフィルターボタンをクリックし、`cart` と入力します。

   ![cart で絞り込み](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-knowledge-graph-3-9628e839f6e147b352e2faa4dd6a8cd3.png)
4.  `cartservice` をクリックし、**Connected Entities** を展開します。**Pod** の横に Pod 数が表示されます。

   ![cart の Connected Entities](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-knowledge-graph-4-d5aaa60e98f16eedd9140411292b5efa.png)

### 方法 2

1.  Entity Catalog の検索バーに `Show Service cartservice` と入力し、候補を選択します。

   ![検索候補](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-knowledge-graph-5-6226478dd67769793ad7bae1dbad0d43.png)
2.  `cartservice` をクリックし、**Connected Entities** を展開すると Pod 数が表示されます。

   ![cart の Connected Entities](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-knowledge-graph-6-62170eec183a9d2ece9b8a129173cc8b.png)

</details>

---

## 質問 4: デプロイ先の Namespace

**すべてのリソースがデプロイされている Namespace はどれですか？**

Knowledge Graph では、Namespace も entity として扱われます。そのため、個々のサービスを開いてラベルを読まなくても、検索バーで直接「すべての Namespace を表示して」と問い合わせられます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**`ecommerce-prod`**

### 方法 1

Entity Catalog を開き、すべてのサービスを表示して、1つずつカーソルを合わせて Namespace を確認します。時間がかかります。

### 方法 2

Entity Catalog の検索バーに `Show all Namespaces` と入力します。`kube-system`、`util-ecommerce-prod`、`ecommerce-prod` のいくつかが表示されます。今回のアプリは `ecommerce-prod` にあります。

</details>

---

<a id="question-5-first-event"></a>

## 質問 5: 最初のイベント

**過去24時間で、問題を引き起こした最初のイベントと思われるものは何ですか？**

productcatalogservice が壊れていることは*わかっています*。Lab 1 で苦労して証明しました。しかし、何が*原因で*壊れたのでしょうか？Lab 1 では、実はこの問いに答えていません。RCA Workbench はまさにこのための機能です。サービスを取り込み、Knowledge Graph に基づいて潜在的な原因を追加し、時間順に並べて上から読んでいきます。一番上にある最初のイベントが、最も早く発火した insight、つまり連鎖的な障害の引き金です。

> **なぜ重要か:** これは Lab 1 のツールでは本当に答えられなかった質問です。従来のアプリが示してくれたのは*何が*壊れているかでした。RCA Workbench は、*何が変わって*問題が始まったのかを教えてくれます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**`productCatalogStopClosingPostgresConnections`** という名前の feature flag が有効化されました。

これは custom insight で、`flagd` サービスのログを監視する recording rule に基づいています。ルールの値が 0 より大きくなる（つまり feature flag の状態が変わる）と、insight が発火します。

**確認方法:**

1. Entity Catalog を開き、`productcatalogservice` を探します。クリックしてたどるか、`productcatalogservice` と入力して **Show Service productcatalogservice** を選択します。

entity を RCA Workbench に追加する方法は2つあります。

**Entity Graph から追加する:**

1. `productcatalogservice` をクリックします。
2. **Troubleshoot in Workbench** ボタンをクリックします。

**結果テーブルから追加する:**

1.  結果テーブルで `productcatalogservice` にカーソルを合わせます。
2.  **Add to Workbench for inspecting** をクリックします。

   ![Workbench に追加](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-1-20cd88a9b3fd507c6b520a0da37c524a.png)
3.  **Open in Workbench** をクリックします。

これで productcatalogservice が RCA Workbench に追加されました。調査してみましょう。

1.  左側の2番目のアイコン **Add potential causes** をクリックします。

   ![Add potential causes](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-2-0a4ec48d8da01e6928fd0df2e0a750e0.png)
2.  **Time** で並べ替えます。

   ![時間順に並べ替え](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-3-ed8aea51e7a06e9292feccdedcea9873.png)
3.  `flagd` が一番上に来ています。この insight が最初に発火しました。ときどき `frontend` の anomaly insight が先に発火することがあります。これは、環境の状況によっては feature flag の insight の発火に1〜2分かかることがあるためです。
4.  `flagd` をクリックし、**amend** カテゴリをクリックします。amend insight は `FeatureFlagStateChange` です。

   ![FeatureFlagStateChange](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-4-7799aab2b6041569b959fab0442b2ba7.png)
5.  さらに詳しく調べるには、`FeatureFlagStateChange` をクリックし、メトリクスのグラフで系列にカーソルを合わせます。

   ![feature flag の詳細](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-5-abbf7e1defbab1d1b3f6a5c18cfd50a9.png)

`productCatalogStopClosingPostgresConnections` feature flag がオンになっていました（その後オフに戻されています）。

### カスタムダッシュボード

`flagd` サービスの KPI drawer には、カスタムダッシュボードが追加されています。`flagd` の横、右端にある **KPI** アイコンをクリックしてください。

![KPI アイコン](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-6-802f5c47ddce94589303fe44f56aa5c7.png)

すべての feature flag とその現在の状態を一覧表示するカスタムダッシュボードが表示されます。

![カスタムダッシュボード](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-7-bb35377d8e933eca2c7ee8c2be02c35d.png)

独自に設定するには、サイドバー → **Configuration** → **KPI display options** を開きます。

![KPI display options](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.5-knowledge-graph-8-1a0d3fc4b30af5ba0ce764a8bbdfcadb.png)

これにより、根本原因の調査のコンテキストを離れることなく、任意のダッシュボードを任意のサービスやサービスの種類に紐付けられます。

**任意 - custom insight の作成方法を確認する。** `{STACK_URL}` はラボ環境の URL に置き換えてください。

- recording rule: `{STACK_URL}/alerting/list?search=flag_state:change`
- custom insight: `{STACK_URL}/a/grafana-knowledge-graph-app/rules/add/file` を開き、`fieldengotelenv.yml` の横にある編集アイコンをクリックします。

</details>

---

<a id="question-6-first-to-error"></a>

## 質問 6: 最初にエラーになったサービス

**過去24時間で、最初にエラーが発生し始めたサービスはどれですか？**

引き金が feature flag だったことはわかりました。では、その結果として*最初に*実際に異常状態になったサービスはどれでしょうか？これは別の質問で、**Summary** タブがその答えを出すためにあります。すべての insight を時間順にフラットに並べた一覧で、誰が最初に壊れたかだけを知りたいときには、Timeline よりずっと読みやすくなっています。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**`productcatalogservice`** です。ただし、表示によっては `productcatalog-postgres` が先に影響を受けているように見えることがあります。この2つはほぼ同時にエラーになるため、メトリクスが scrape されるタイミングによって順序が入れ替わることがあります。また、feature flag の insight の発火に1〜2分かかることがあるため、ときどき `frontend` の anomaly insight が先に発火することもあります。

**確認方法:**

1.  前の質問の続きから進めます。潜在的な原因が追加された状態の RCA Workbench が開いているはずです。
2.  （前と同様に）**Time** で並べ替えます。
3.  **Summary** タブをクリックします。こちらのほうがずっと読みやすくなります。

   ![Summary タブ](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.6-knowledge-graph-0883c466f705847c21aba3bf545b92d6.png)

> **注記**

   Summary は一覧が短いときに最も効果的です。insight やサービスが多いと、すべてが一度に表示されるため圧倒されてしまうことがあります。**Timeline** では見たい部分を展開・折りたたみできますが、Summary では常にすべてが表示されます。

4.  正確な順序は異なる場合がありますが、次のような insight が表示されます。
   - `productcatalog-postgres` は `PostgresSQLHighConnections` の failure insight と、いくつかの anomaly insight を出しています。
   - `productcatalogservice` は、`oteldemo.ProductCatalogService/ListProducts` エンドポイントで `ErrorRatioBreach` の error insight を、また `KubePodCrashLooping` の failure insight を出しています。
   - `frontend` には、すべてのリクエストが通過するため、anomaly insight と `ErrorRatioBreach` insight が表示されます。frontend は recommendationservice に問題が発生していることも示しています。これを表示するには、**Timeline** に戻り（Summary は読み取り専用です）、frontend サービスで **Add potential causes** をクリックします。

</details>

---

## 質問 7: Redis のバージョン *（任意）*

**デプロイされている Redis のバージョンは何ですか？**

インシデントが解決したら、post-mortem のために詳細を記録しておきたくなります。Knowledge Graph のすべての entity は、バージョン、イメージ、ラベルなどのプロパティを持っており、カタログを離れずに直接確認できます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**8.2.1**

**確認方法:**

-  メインの検索バーに `redis` と入力し、**Show Service redis** を選択します。

   ![redis を検索](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.7-knowledge-graph-1-f3751adc2182dfb555f5f09bb2cf8a8c.png)
-  entity にカーソルを合わせると、バージョンが `8.2.1` と表示されます。

   ![Redis のバージョン](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.7-knowledge-graph-2-31283e900753dd98fc04cdbb9f85eff0.png)
-  または、**Show more** をクリックすると完全なプロパティページが開き、バージョンやその他のプロパティを確認できます。

   ![完全なプロパティページ](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.7-knowledge-graph-3-80f94ca61af6dce98c8fba9f5cc1f5b1.png)

</details>
