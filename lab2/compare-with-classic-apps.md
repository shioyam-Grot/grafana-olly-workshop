<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab2/compare-with-classic-apps -->

# 2.2. 従来の Observability アプリと比較する

*Knowledge Graph を使って、いくつかの質問に数クリックで答えられました。今度は同じ質問に Lab 1 のツール（App O11y、K8s Monitoring）だけで答えてみて、かかった時間を計ってみましょう。*

従来のアプリでもこれらの質問にはすべて答えられますし、うまく答えられます。従来のアプリを置き換えようという話ではありません。ポイントは*どのように*答えるかです。いくつのパネルを読み込む必要があるか、いくつのタブを切り替えるか、どれだけのコンテキストを頭の中に保持しておく必要があるか。Knowledge Graph はまさにこの違いを解消するために作られています。

> **情報**
>
> 以下の解説は、参考として Knowledge Graph を使う方法で説明しています。まずは従来のアプリを使う方法で自分で試してから、答えを開いてください。

---

## 質問 1: 割り当てられたメモリ

**`productcatalogservice` workload に割り当てられているメモリはどのくらいですか？**

Lab 1.3 では、K8s Monitoring の workload ビューまでクリックしてたどり、この質問に答えました。Knowledge Graph では、同じ Workload Memory パネルがサービスの KPI drawer を開くだけで表示されます。アプリを切り替える必要はありません。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**100 MiB**

**確認方法（Knowledge Graph を使う場合）:**

1. 検索バーで `productcatalogservice` サービスを表示するか、Entity Catalog → **Show all services** を開いて `productcatalogservice` で絞り込みます。
2. **KPI drawer** を開きます。
3. **Kubernetes** タブをクリックします。
4. **Workload Memory** パネルで **memory limits** を確認します。

![workload のメモリ](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.1-knowledge-graph-747f787283a6a056f63a4e45ddbc487e.png)

</details>

---

## 質問 2: エラー率

**過去24時間で、`productcatalogservice` のエラー率は最大何パーセントに達しましたか？**

Lab 1.2 では、Application Observability のサービス詳細ページに移動し、Errors パネルにカーソルを合わせました。KPI drawer の **Service Overview** タブはその画面とまったく同じ内容で、すでに開いている Knowledge Graph の entity 画面からそのまま開けます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**15%〜40%** です。

**確認方法（Knowledge Graph を使う場合）:**

1. 検索バーで `productcatalogservice` を表示するか、Entity Catalog → **Show all services** を開いて絞り込みます。
2. **KPI drawer** を開きます。
3. **Service Overview** タブをクリックします。
4. **Errors** パネルで、スパイクがどこまで上がっているかを確認します。

</details>

---

## 質問 3: エラーメッセージ

**過去24時間で、`productcatalogservice` がエラーになるとき、どのようなエラーメッセージを出していますか？**

たどる経路を比べてみましょう。Lab 1.2 では、Services 一覧 → サービス詳細 → Errors パネル → Traces ボタン → トレースを選択 → span を展開 → イベントを展開、という順にたどりました。エラーメッセージを見るまでに6回のクリックが必要です。RCA Workbench ならこれを短縮できます。insight のバーにカーソルを合わせ、その insight の時間帯に絞り込まれたログやトレースに直接移動できます。コンテキストを踏まえた動作をするので、insight から「Logs」をクリックすると、ログはすでにその insight に関連する時間範囲とサービスで絞り込まれています。フィルターを適用しているのはあなたではなく、Workbench です。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

- **トレースから調べる場合:** `pq: sorry, too many clients already`
- **ログから調べる場合:**
   - `panic: runtime error: invalid memory address or nil pointer dereference`
   - `pq: sorry, too many clients already`

ここで注目したいのは、**コンテキスト駆動の RCA Workbench** を使う方法です。

1.  `productcatalogservice` を追加した RCA Workbench を開き、接続されているサービスを追加します（[2.1 の最初のイベントの質問](knowledge-graph.md#question-5-first-event)を参照）。
2.  **Summary** タブをクリックします。Timeline でも構いませんが、Summary のほうが読みやすくなります。[2.1](knowledge-graph.md#question-6-first-to-error) で触れたとおり、Summary は一覧が短いときに最も効果的です。insight が多い場合は、見たい部分を展開・折りたたみできる Timeline を使いましょう。
3.  `oteldemo.ProductCatalogService/ListProducts - ErrorRatioBreach` insight の赤いバーにカーソルを合わせます。

   ![ErrorRatioBreach にカーソルを合わせた状態](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-knowledge-graph-1-31501aa9b385e2f4a47dcfdd07baf069.png)
4.  **Logs** をクリックします。KPI drawer の Logs タブが、その insight の時間帯で開き、すべてのエラーメッセージが表示されます。`pq: sorry, too many clients already` と `panic: runtime error: invalid memory address or nil pointer dereference` が確認できます。

   ![ログ画面](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-knowledge-graph-2-05347edba8e45e35ed21c5533efaa967.png)
5.  KPI drawer を閉じます。
6.  再度赤いバーにカーソルを合わせ、**Traces** をクリックします。KPI drawer の Traces タブが同じ時間帯で開きます。エラーのラジオボタンを選択し、エラーになったトレースだけを表示します。

   ![エラーになったトレース](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-knowledge-graph-3-81d13082c5eb2c9eb6ad967e7ad5e949.png)
7.  任意のトレースをクリックし、赤い丸が付いた `productcatalogservice` の span をクリックします。

   ![span の詳細](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-knowledge-graph-4-8f50449e8bbfb9bb1ea2371414793803.png)
8.  その span のイベントを展開すると、`pq: sorry, too many clients already` が表示されます。

   ![span のイベント](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-knowledge-graph-5-1dd89c864ddd92ccd8cd63a923a2325f.png)

</details>

---

## 質問 4: エラーが発生しているエンドポイント

**過去24時間で、`productcatalogservice` の中でエラーが発生しているエンドポイントはどれですか？**

Lab 1.2 と同じ **Operations** パネルの考え方ですが、entity の画面を離れることはありません。このパネルはサービスの KPI drawer 内に、先ほど見た workload のメモリデータと並んで表示されます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**`oteldemo.ProductCatalogService/ListProducts`**

**確認方法:**

1. 検索バーで `productcatalogservice` を表示するか、Entity Catalog → **Show all services** を開いて絞り込みます。
2. **KPI drawer** を開きます。
3. **Service Overview** タブをクリックします。
4. **Operations** パネルを確認します。`oteldemo.ProductCatalogService/ListProducts` の sparkline に目に見える変動があります。

![Operations の sparkline](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.4-knowledge-graph-639cc7d2bad4659bce5983420cc013b5.png)

</details>
