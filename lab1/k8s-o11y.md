<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab1/k8s-o11y -->

# 1.3. Kubernetes Monitoring

*Frontend Observability で、ECサイトが 500 エラーを返していることがわかりました。Application Observability で、原因を `productcatalogservice` に絞り込みました（このデモでは意図的に障害を起こしています）。さらに一段下のレイヤーに降りて、その Pod で実際に何が起きているのかを Kubernetes に聞いてみましょう。*

Grafana Cloud の [Kubernetes Monitoring](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/) は、クラスター（workload、Pod、コンテナ、ノード）を一元的に把握できるビューです。構築済みのダッシュボード、アラート、right-sizing の推奨値がすべて1か所にまとまっています。アプリを切り替える必要も、PromQL を書く必要もありません。

Grafana で **Kubernetes Monitoring** アプリを開いてください。

![Kubernetes Monitoring - はじめに](https://grafana.github.io/observability-workshop-breakouts/assets/images/0.3-k8s-instructions-424b45e946c8129e91d56323b7e1a02b.png)

---

<a id="question-1-pod-restarts"></a>

## 質問 1: Pod の再起動

**`productcatalogservice` は何回再起動していますか？**

コンテナが panic したり liveness probe に失敗したりすると、Kubernetes はそのコンテナを終了させ、新しいコンテナを起動します。この再起動回数は、アプリケーション層の何かがプロセスを落としていることを最も直接的に示すシグナルです。Frontend Observability のセクションでフロントエンドが返していた 500 エラーは、この数字が増えていることの下流で起きている現象です。productcatalogservice が不健全であることはすでにわかっています。問題は、App O11y で見たのと同じ状況が K8s 側でも見えているかどうかです。

答えにたどり着く方法は2つあります。Application Observability 経由（今いる場所）か、Kubernetes Monitoring アプリから直接確認する方法です。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

正確な回数は変動します。このサービスは（意図的に）常に障害を起こしているためです。次のいずれかの方法で現在の値を確認してください。

### 方法 1 - Application Observability から確認する

1. Application Observability アプリを開きます。
2. **Services** タブが選択されていることを確認します。
3. `productcatalogservice` をクリックします。
4. **Kubernetes** ボタンをクリックし、`k8s.pod.name` の項目を探します。
5. **Pod information** の下にある **number of restarts** の項目を確認します。

![再起動回数が表示された Pod information](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-1-cd9ca6bbc93906c88f8e3b7be66f5d81.png)

### 方法 2 - Kubernetes Monitoring から確認する

1.  **Observability** メニューから Kubernetes Monitoring アプリを開きます。
2.  **Workloads** をクリックします。

   ![Workloads](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-2-213e34b93a9859d92da4894e9a5b7a9f.png)
3.  一覧をスクロールするか、上部のフィルターで絞り込みます。

   ![フィルター後の Workloads](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-3-1c89baaea9bf12224339c50de81f0c51.png)
4.  `europe` を**含まない**ほうの `productcatalogservice` をクリックします。Deployment の概要が開きます。**pods** テーブルの Pod 名をクリックすると、Pod レベルのデータが表示されます。

   ![workload の詳細](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-4-2192056ec78fc5b475088efe7a7b9c27.png)
5.  **Pod information** の下にある **number of restarts** の項目を確認します。

   ![Pod information](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.1-k8s-o11y-1-cd9ca6bbc93906c88f8e3b7be66f5d81.png)

</details>

---

## 質問 2: イメージとバージョン

**`productcatalogservice` で使われているイメージ名とバージョンは何ですか？**

Pod は crash loop しています。次に対応者が必ず尋ねるのは「*実際にはどのバージョンが動いているのか？*」です。コンテナイメージのタグを見れば、「さっきリリースした変更による退行だ」なのか「何週間も前から壊れていたのに誰も気づかなかった」なのかがわかります。この情報は Pod に直接記録されています。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

```text
us-central1-docker.pkg.dev/field-engineering-grafana/application-environment/productcatalogservice:v{VERSION}
```

`{VERSION}` はデモアプリの更新に伴って変わることがあります。

**確認方法:**

1. productcatalogservice の Pod ビューを開きます（[再起動の質問](#question-1-pod-restarts)を参照）。
2. **Containers** パネルの **IMAGE SPEC** 列を読み取ります。

![イメージの仕様](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.2-k8s-o11y-1-1f0458ee2f2615156da4166a22598eee.png)

</details>

---

## 質問 3: 割り当てられたメモリ

**`productcatalogservice` に割り当てられているメモリはどのくらいですか？**

すべての workload は、デプロイ時にメモリの request と limit を宣言します。これは Kubernetes が強制するガードレールです。Pod のサイズが適切かどうかを判断する前に、まずどれだけ要求しているかを知る必要があります。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**100 MiB**

**確認方法:**

1. productcatalogservice の workload ビューまたは Pod ビューを開きます（[再起動の質問](#question-1-pod-restarts)を参照）。
2. **Workload Memory** パネルの破線にカーソルを合わせます。

![workload のメモリ](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.3-k8s-o11y-3bf28a32aab41af147c077352304c423.png)

</details>

---

<a id="question-4-right-sizing"></a>

## 質問 4: right-sizing

**`productcatalogservice` のメモリは oversized ですか、それとも undersized ですか？**

この workload には 100 MiB が割り当てられていますが、実際にそれに近い量を使っているのでしょうか？Grafana Kubernetes Monitoring は[要求量と実際の使用量を追跡](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/optimize-resource-usage/)し、その差を色付きの gauge で表示します。ひと目見れば答えがわかります。

> **なぜ重要か:** 過剰にプロビジョニングされた workload は、他の workload が使えたはずのキャパシティを占有してしまいます。Grafana のドキュメントによると、right-sizing のデータは「チームが request を引き下げ、既存のハードウェア上の未使用キャパシティを取り戻すことを可能にします」。つまり、新たに購入しなくて済むキャパシティ（とコスト）です。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**oversized です。**

**確認方法:**

1.  productcatalogservice の Pod ビューを開きます（[再起動の質問](#question-1-pod-restarts)を参照）。
2.  **Containers** テーブルの `productcatalogservice` コンテナをクリックします。

   ![Containers テーブル](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-1-84908dd330242ef6a4bd57fac5801153.png)
3.  現在のサイズは 100 MiB ですが、gauge には赤色で 81 MiB と表示されています。これは oversized であることを意味します。

   ![コンテナの メモリ gauge](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-2-71e7b5b0ea4b8fd5dca7b88ec29d48f0.png) ![コンテナのメモリ詳細](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-3-4a45120d60f45fbf2f747261de2a169b.png)

</details>

---

## 質問 5: メモリの推奨値

**`productcatalogservice` のメモリの推奨値はいくつですか？**

workload を oversized と判定したのと同じ画面で、具体的にいくつに設定すべきかもわかります。推測も計算も不要です。実際の使用量から導き出された数値なので、そのまま Helm chart や kustomization に反映できます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**15 MiB〜20 MiB** です。

**確認方法:**

1.  [前の質問](#question-4-right-sizing)の続きから確認します。

   ![メモリ gauge](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.4-k8s-o11y-2-71e7b5b0ea4b8fd5dca7b88ec29d48f0.png) ![メモリの推奨値](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.5-k8s-o11y-aacc098bd1b21f373e4162a58447dc25.png)

</details>

---

## 質問 6: クラスターのキャパシティ *（任意）*

**K8s クラスターに割り当てられているメモリと CPU はどのくらいですか？**

個々の workload から視点を引いてみましょう。**Clusters** ビューには、管理しているすべてのクラスターの物理キャパシティが表示されます。これは、他のすべてがその範囲内に収まらなければならない上限です。workload をスケールアップする余地が本当にあるのか、すでに上限に達しているのかを知りたいときに役立ちます。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**CPU:** 8 コア
 **メモリ:** 15 GB

**確認方法:**

1.  **Observability** メニューから Kubernetes Monitoring アプリを開きます。
2.  **Clusters** をクリックします。

   ![Clusters](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.6-k8s-o11y-1-b727b4c98c559ee1f36f704d19e710e2.png)
3.  クラスターをクリックします。
4.  **Cluster CPU** パネルと **Cluster Memory** パネルの両方で、**Physical capacity of cluster** の線を確認します。

   ![クラスターのキャパシティ](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.6-k8s-o11y-2-7a38c23dfa70e04559a0c50e39d4cf1e.png)

</details>

---

## 質問 7: クラスターのエネルギー使用量 *（任意）*

**K8s クラスターの総エネルギー使用量はどのくらいですか？**

クラスターレベルにいるついでに、もう1つ確認しましょう。Kubernetes Monitoring は[フリート全体のエネルギー使用量](https://grafana.com/docs/grafana-cloud/monitor-infrastructure/kubernetes-monitoring/)も表示します。チームの誰かがリソース効率と合わせてサステナビリティの指標を追跡している場合でも、データはすでに揃っています。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

約 **200 Wh** です。

**確認方法:**

1.  **Observability** メニューから Kubernetes Monitoring アプリを開きます。
2.  **Clusters** をクリックします。

   ![Clusters](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.6-k8s-o11y-1-b727b4c98c559ee1f36f704d19e710e2.png)
3.  クラスターをクリックします。
4.  **Energy** タブをクリックします。
5.  `Total` 系列を確認します。他の系列に隠れている場合があるので、凡例を確認してください。

   ![エネルギー使用量](https://grafana.github.io/observability-workshop-breakouts/assets/images/3.7-k8s-o11y-c84390ef1eddd603f1cd79988d2a8f91.png)

</details>
