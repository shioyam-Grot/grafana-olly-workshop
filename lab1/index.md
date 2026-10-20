<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab1/ -->

# Lab 1 - Grafana Observability 入門

このラボでは、先ほど紹介した Grafana Cloud の Observability アプリを使ってトラブルシューティングのシナリオに取り組み、各ツールを実際に操作して体験します。

## 手順

スライドに記載された URL にアクセスしてください。次の機能が使える Grafana Cloud スタックにアクセスできます。

1. Frontend Observability
2. Application Observability
3. Kubernetes Monitoring

質問に取り組む時間は20分です。

> **ヒント**
>
> まずは自分で各質問に挑戦してみてください。各質問の下にある **答えを表示** ブロックを開くと、手順の解説とスクリーンショットが表示されます。

- **[1.1. Frontend Observability](frontend-o11y.md)**: あなたはオンコール担当です。顧客から、ECサイトの調子がおかしい（ページの読み込みが遅い、ときどきエラーが出る）という報告が届いています。どこから調べ始めますか？ユーザーと同じ場所、つまりブラウザからです。

- **[1.2. Application Observability](app-o11y.md)**: Frontend Observability から、ECサイトのすべてのページで 500 エラーが発生していることがわかりました。ブラウザで見えているのは症状であり、原因はサーバー側にあります。バックエンドに踏み込んで、どのサービスが問題なのかを突き止めましょう。

- **[1.3. Kubernetes Monitoring](k8s-o11y.md)**: Frontend Observability で、ECサイトが 500 エラーを返していることがわかりました。Application Observability で、原因を productcatalogservice に絞り込みました（このデモでは意図的に障害を起こしています）。さらに一段下のレイヤーに降りて、その Pod で実際に何が起きているのかを Kubernetes に聞いてみましょう。
