<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab1/app-o11y -->

# 1.2. Application Observability

*Frontend Observability から、ECサイトのすべてのページで 500 エラーが発生していることがわかりました。ブラウザで見えているのは症状であり、原因はサーバー側にあります。バックエンドに踏み込んで、どのサービスが問題なのかを突き止めましょう。*

Grafana Cloud の [Application Observability](https://grafana.com/docs/grafana-cloud/monitor-applications/application-observability/) は、アプリケーションの問題に対する平均修復時間（MTTR）の短縮を支援する APM ソリューションです。OpenTelemetry、Grafana Alloy、構築済みのダッシュボードを活用し、スタック内のすべてのサービスを1か所で可視化します。

Grafana で **Application Observability** アプリを開いてください。

![Application Observability - はじめに](https://grafana.github.io/observability-workshop-breakouts/assets/images/0.2-app-instructions-7646e4bfc3a9fa3fa80668402e0b2561.png)

---

## 質問 1: サービスの言語

**このデモアプリではどの言語が使われていますか？**

トラブルシューティングの前に、何を見ているのかを把握しておくと役立ちます。**Service Inventory** には、アプリ内で OpenTelemetry によって計装されたすべてのサービスが一覧表示され、ランタイムの言語も自動検出されます。ひと目見るだけで、スタックがどれだけ多様な技術で構成されているかがわかります。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

かなりたくさんあります。

- Go
- Java
- .NET
- C++
- Ruby
- Erlang
- JavaScript
- PHP
- Python
- Rust

**確認方法:**

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. **Technology** 列のアイコンを確認します。どのアイコンかわからない場合は、カーソルを合わせてください。

![Technology アイコンが表示された Services タブ](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.1-app-o11y-86fb8ebb05dd1bcc5f9a2b9f0ffa2d21.png)

</details>

---

## 質問 2: エラー率

**過去1時間で、`productcatalogservice` のエラー率は最大何パーセントに達しましたか？**

Frontend O11y のエラーには `/api/products` が含まれていました。これは `productcatalogservice` が関係していることを強く示唆しています。このサービスをクリックすると、上部にリクエストレート、エラー率、レイテンシが表示されます。何かおかしいと感じたときに最初に確認する3つのシグナルです。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**20%〜100%** の範囲で変動します。

**確認方法:**

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. `productcatalogservice` をクリックします。
4. **Errors** パネルのピークにカーソルを合わせます。

![Errors パネル](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.2-app-o11y-60072bbc2302e294bac7a6ca8291f805.png)

</details>

---

## 質問 3: エラーメッセージ

**`productcatalogservice` がエラーになるとき、どのようなエラーメッセージを出していますか？**

20〜100% のエラー率は深刻です。次は実際の例外を確認する必要があります。方法は2つあります。トレースの waterfall から（失敗した1件のリクエストを掘り下げる）か、サービスのログからです。どちらも異なる角度から同じ根本原因にたどり着くはずです。トレースとログは同じインシデントを別の視点から見たものです。トレースは「このリクエストが失敗し、その経路はこうだった」ことを教えてくれ、ログは「プロセスが出力したすべての内容」を教えてくれます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

どちらの方法で調べるかによって、答えがいくつかあります。

- **トレースから調べる場合:** `pq: sorry, too many clients already`
- **ログから調べる場合:**
   - `panic: runtime error: invalid memory address or nil pointer dereference`
   - `pq: sorry, too many clients already`

**確認方法:**

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. `productcatalogservice` をクリックします。

### 方法 1 - トレースから調べる

1.  **Errors** パネル内の **Traces** ボタンをクリックします。

   ![Errors パネル - Traces ボタン](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-app-o11y-1-4b5073430624b09bdc316e08174a9630.png)
2.  表内の任意の **Trace ID** をクリックします。どれも productcatalogservice のエラートレースです。

   ![span の詳細](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-app-o11y-2-e5c8a61b25c4791246dd604aa5c984be.png)
3.  トレースの waterfall が表示されます。横に赤い丸が付いている `productcatalogservice` の span をクリックします。
4.  **Events (2)** をクリックし、両方のイベントを展開します。

   ![イベント](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-app-o11y-3-86811c15a2afd4c2c7ea13bd4b61a400.png)
5.  `exception` イベントと `error` イベントの両方に、値 `pq: sorry, too many clients already` が入っています。

   ![Logs タブ](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-app-o11y-4-0ffcaac4b43b01f3e14f08f2500cebab.png)

### 方法 2 - ログから調べる

1.  上部の **Logs** タブをクリックし、このサービスが出力したすべてのログを表示します。
2.  `pq: sorry, too many clients already` のログが大量に表示され、さらに下に `panic: runtime error: invalid memory address or nil pointer dereference` を含むスタックトレースがあります。

   ![ログ](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.3-app-o11y-5-c6f2a5195877e7fcc1f91359ed4aa2e5.png)

</details>

---

## 質問 4: エラーが発生しているエンドポイント

**`productcatalogservice` の中で、エラーが発生しているエンドポイントはどれですか？**

`pq: sorry, too many clients already` は Postgres の接続枯渇エラーです。サービスがデータベース接続を使い果たしています。しかし、これはすべてのエンドポイントで起きているのでしょうか、それとも1つだけでしょうか？**Operations** パネルでは、サービスのトラフィックがエンドポイントごとに分けて表示されるので、発生箇所を特定できます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

`oteldemo.ProductCatalogService/ListProducts`

**確認方法:**

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. `productcatalogservice` をクリックします。
4. **Operations** パネルで、**Errors** 列が 0 以外になっている唯一のエンドポイントを探します。

![エラーが発生しているエンドポイント](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.4-app-o11y-2b2bd91be4e8f0fc572c23aa77d75e3f.png)

</details>

---

## 質問 5: サービス間の連携

**`recommendationservice` と連携しているサービスはどれですか？**

ついでに全体像も把握しておきましょう。recommendationservice は、呼び出しグラフ上で productcatalogservice の隣にあります。何がこのサービスを呼び出し、このサービスは何を呼び出しているのでしょうか？サービスの詳細画面では、これが **Inbound**（呼び出し元）と **Outbound & databases**（依存先）に分けて表示されます。また、[Service map](https://grafana.com/docs/grafana-cloud/monitor-applications/application-observability/) タブでは同じ情報をグラフで確認できます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

次の3つです。

- **Outbound & databases:**
   - `flagd`
   - `productcatalogservice`
- **Inbound:**
   - `frontend`
   - `chatservice`

**確認方法:**

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. `recommendationservice` をクリックします。
4. downstream のサービスは **Outbound & databases** パネルに表示されます。
5. upstream のサービスは **Inbound** パネルに表示されます。

![サービス間の連携](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.5-app-o11y-1-e5b8b6efc939a5a968a352f6ab41921d.png)

**Service map** タブからも確認できます。

![Service map](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.5-app-o11y-2-50ed8069331ebe6ffd0371e593e915e0.png)

</details>

---

## 質問 6: Pod 名 *（任意）*

**productcatalogservice を実行している Pod の名前は何ですか？**

Application Observability は、`k8s.pod.name` などの属性を使って OTel のテレメトリを Kubernetes に結び付けています。これにより、「サービス X が壊れている」から「調べるべきは Pod Y だ」へとすぐに移れます。ここで Pod 名を控えておきましょう。次のラボへの橋渡しとして使います。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

Pod が再起動するたびに名前は変わりますが、`productcatalogservice-...` で始まります。たとえば `productcatalogservice-58dffdf697-mjpmw` のような名前です。

**確認方法:**

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. `productcatalogservice` をクリックします。
4. **Kubernetes** ボタンをクリックし、`k8s.pod.name` の項目を探します。

![Pod 名](https://grafana.github.io/observability-workshop-breakouts/assets/images/2.6-app-o11y-29defb14d258aa09d1702ce42052915a.png)

</details>
