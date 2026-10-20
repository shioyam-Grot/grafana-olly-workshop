<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/frontend-o11y -->

# 3.1. Frontend Observability のデータをクエリする

*ここまでで、productcatalog の謎を2通りの方法で解きました。1つは Lab 1 のアプリを使う方法、もう1つは Lab 2 の Knowledge Graph を使ったより短い方法です。3つ目の選択肢があります。どこにも移動せず、ただ聞くだけです。*

[Grafana Assistant](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/) は、Grafana に組み込まれた専用の LLM です。運用担当者、開発者、SRE が、自然言語でトラブルシューティング、ダッシュボードの管理、製品に関する質問への回答を行えます。このセクションでは、Lab 1.1 で調べたのと同じ `ecommerce` フロントエンドアプリについて同じ質問をし、Assistant がどのように答えを導くかを見ていきます。

基本の質問では、フロントエンドのページロードとエラーを扱います。追加課題では、`@` コンテキストを使って Assistant を特定の data source やラベルに固定する方法を学びます。あらゆる場所を検索させるのではなく、テレメトリの特定の部分に集中させたいときに便利です。

---

## 質問 1: ページロード数

**フロントエンドアプリ `ecommerce` について、過去3時間のページロード数はいくつですか？**

Lab 1.1 と同じ最初の質問です。最小限のコンテキストしか与えなかったときに Assistant がどう動くかを確認するのにちょうどよい質問です。`@` メンションも data source のヒントもなしで、質問だけを送ります。Assistant が答えるためにどの tool call を選ぶかに注目してください。tool call を展開すれば、「信じてください」ではなく「これが作業の過程です」という状態になり、結果を信頼する前に正しいデータをクエリしたかを検証できます。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

過去3時間で約 **1,300〜1,780 リクエスト/秒** です。

Assistant が件数そのものではなく*レート*（リクエスト/秒）で答えている点に注意してください。tool call を展開して、実際に何をクエリしたのかを確認すべき理由がよくわかる例です。

![Assistant のページロード数に関する回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.1-grafana-assistant-1-4df5cd02ec8780fea6f16277dd1d53e3.png)

</details>

---

## 質問 2: ページのエラー

**それらのページにエラーはありますか？**

同じ会話を続けると、Assistant はすでに把握している内容を引き継げます。これは、同じサービス詳細ページにとどまっているのと同じことを AI で行っている状態です。改めて伝えなくても、前のプロンプトの `ecommerce` アプリを覚えているはずです。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

はい。Assistant は、前のプロンプトの `ecommerce` アプリを引き継いだうえで、Lab 1.1 で確認したのと同じページ（`/`、`/cart`、`/cart/checkout/*`、`/product/*`）でエラーが発生していると報告します。

![Assistant のエラーに関する回答](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.2-grafana-assistant-71b5351673e6e960dda9f193cc8a6fc4.png)

</details>

---

## 質問 3: ログのコンテキスト *（任意）*

**Frontend Observability について別の質問をしてみましょう。今回はログの data source を `@` コンテキストで指定します。**

ここまでは、Assistant が自由に data source を選んでいました。`@` 構文を使えば、あなたが主導権を握れます。特定のリソース（ここではログの data source `grafanacloud-*****-logs`）を指定すると、Assistant は推測せずにそこでクエリを実行します。正しい答えが特定の場所にあるとわかっているときは、いつでもこの方法を使いましょう。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

`@` コンテキストを設定すると、Assistant は自分で data source を選ぶのではなく、固定されたログの data source（`grafanacloud-*****-logs`）に対してクエリを実行します。tool call を展開して、どの data source が使われたかを確認してください。

![ログの data source をコンテキストに指定した Assistant](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-grafana-assistant-8e96bc2bc3c812e67771767d7e950c3e.png)

</details>

---

## 質問 4: ラベルのコンテキスト *（任意）*

**Frontend Observability について別の質問をしてみましょう。今回は `app_id` ラベルを `@` コンテキストで指定します。**

Assistant をさらに細かく固定することもできます。data source だけでなく、その中の特定のラベル値にまで絞り込めます。Frontend Observability では `app_id` ラベルを使って各ログ行がどのアプリのものかを識別しているので、Assistant を1つの `app_id` に固定すると、そのフロントエンドだけに絞り込めます。

> **試してみよう:** 答えを開く前に、Grafana Assistant に聞いてみましょう。

<details>
<summary>答えを表示</summary>

コンテキストメニューで **Labels** を開きます。

![Labels メニュー](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.4-grafana-assistant-1-3fd6a9947c5e14d05dc7e8c7abd8856f.png)

ログの data source を選択します。

![ログの data source](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.4-grafana-assistant-2-b2571c45630d86bf2bc2ebb6d588e9ae.png)

`app_id` ラベルを選択します。

![app_id ラベル](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.4-grafana-assistant-3-e7cb10bfe9d620208e9910141430eedf.png)

app id を選択します。

![app id の選択](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.4-grafana-assistant-5-f48622cf91a3570ea6a0b16185ccfc55.png)

</details>
