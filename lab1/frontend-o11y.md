<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab1/frontend-o11y -->

# 1.1. Frontend Observability

*あなたはオンコール担当です。顧客から、ECサイトの調子がおかしい（ページの読み込みが遅い、ときどきエラーが出る）という報告が届いています。どこから調べ始めますか？ユーザーと同じ場所、つまりブラウザからです。*

Grafana Cloud の [Frontend Observability](https://grafana.com/docs/grafana-cloud/monitor-applications/frontend-observability/) を使うと、その視点で状況を把握できます。[Faro Web SDK](https://grafana.com/docs/grafana-cloud/monitor-applications/frontend-observability/instrument/) を使ってブラウザから直接テレメトリ（パフォーマンスメトリクス、エラー、ログ、クライアントサイドのトレース）を収集するため、さまざまなデバイス、ブラウザ、ネットワークで実際に何が起きているのかを確認できます。

Grafana で **Frontend Observability** アプリを開き、`ecommerce` フロントエンドアプリをクリックしてください。

![Frontend Observability - はじめに](https://grafana.github.io/observability-workshop-breakouts/assets/images/0.1-frontend-instructions-9575a403b362a518506f05f2ef80c18d.png)

---

## 質問 1: ページロード数

**過去1時間のページロード数はいくつですか？**

問題を探しに行く前に、まずはベースラインを把握しましょう。今、アプリはどのくらい忙しいでしょうか？アプリ概要にある **Page Loads** パネルは、トラフィックをひと目で確認できる指標です。最初に知っておくべき数字です。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**500〜700** ページロードです。

**確認方法:**

1. Frontend Observability アプリを開きます。
2. `ecommerce` アプリを選択します。
3. **Page Loads** パネルの値を読み取ります。

![Page Loads パネル](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.1-frontend-o11y-4b6bffe8fb566546a8dfd748a1e6896a.png)

</details>

---

## 質問 2: Largest Contentful Paint

**過去1時間の Core Web Vitals のひとつ、Largest Contentful Paint (LCP) の値はいくつですか？**

トラフィックは問題ありません。では、ページは実際にどのくらい速く感じられるでしょうか？LCP は、Frontend Observability がアプリ概要に直接表示する Core Web Vitals のひとつです。ページの読み込みが十分に速く、ユーザーが離脱せずにいられるかを Google が測定する指標です。ページロードのメトリクスは、ユーザーの不満を先取りして示す指標です。遅いページは離脱され、離脱されたページは失注につながります。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

**1秒〜2秒** です。

**確認方法:**

1. Frontend Observability アプリを開きます。
2. `ecommerce` アプリを選択します。
3. **LCP** パネルの値を読み取ります。

![LCP パネル](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.2-frontend-o11y-ddc9b994d72514c5e6fed10ff79b3460.png)

</details>

---

## 質問 3: エラーが発生しているページ

**過去1時間で、エラーが発生しているページはどれですか？**

トラフィックは健全で、LCP も妥当な値です。しかし、顧客が問い合わせてきているのはエラーについてです。**Page Performance** パネルはリクエストをページごとにまとめて表示するので、*どこで*問題が起きているのかがすぐにわかります。すべてのページでエラーが出ていれば、スタックの深い部分に原因があることを示しています。1ページだけなら、その時点で範囲を絞り込めたことになります。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

4ページすべてです。

- `/`
- `/cart`
- `/cart/checkout/*`
- `/product/*`

**確認方法:**

1. Frontend Observability アプリを開きます。
2. `ecommerce` アプリを選択します。
3. **Page Performance** パネルで、各ページ ID のエラー列を確認します。すべてのページでエラーが表示されています。

![Page Performance パネル](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.3-frontend-o11y-79bbef75411859a214f9b66f23b06b4f.png)

</details>

---

## 質問 4: エラーの例 *（任意）*

**過去1時間に発生したエラーの例を3つ挙げてください。**

すべてのページが壊れています。これは、問題がフロントエンドの特定のルートではなく、バックエンドで起きていることを強く示しています。Frontend Observability を離れる前に、ブラウザから送られてきている実際の例外メッセージをいくつか確認しておきましょう。これらのエラーは、次のラボに持ち込む手がかりになります。

> **試してみよう:** 下の答えを開く前に、Grafana で答えを探してみましょう。

<details>
<summary>答えを表示</summary>

`{...}` の部分には任意の値が入ります。組み合わせが多すぎるため、すべては列挙できません。

- `Response not ok. Status code: 500. Status text: ''. Url: {YOUR_APP_URL}/api/data?contextKeys=telescopes`
- `Response not ok. Status code: 500. Status text: ''. Url: {YOUR_APP_URL}/api/products?currencyCode={CURRENCY_CODE}`
- `Response not ok. Status code: 500. Status text: ''. Url: {YOUR_APP_URL}/api/recommendations?productIds={PRODUCT_ID}&sessionId={SESSION_ID}&currencyCode={CURRENCY_CODE}`

**確認方法:**

1. Frontend Observability アプリを開きます。
2. `ecommerce` アプリを選択します。
3. **Errors** タブをクリックします。
4. **Top Errors** パネルを確認します。

![Top Errors パネル](https://grafana.github.io/observability-workshop-breakouts/assets/images/1.4-frontend-o11y-1216b00384c2221e8240d2040c2bf6f7.png)

</details>
