<!-- Source: https://grafana.github.io/observability-workshop-breakouts/lab3/ -->

# Lab 3 - AI を活用した Observability

このラボでは、Lab 1 のトラブルシューティングのシナリオに再び取り組みますが、今回は AI、具体的には [Grafana Assistant](https://grafana.com/docs/grafana-cloud/machine-learning/assistant/) を使って答えを見つけます。

## 手順

**Lab 1 と同じ環境**を使用します。スライドに記載された Lab 1 環境の URL にアクセスしてください。次の機能が使えます。

1. Frontend Observability
2. Application Observability
3. Kubernetes Monitoring

右上の **Grafana Assistant** ボタンをクリックして始めましょう。コンテキストを引き継ぐため、同じセクション内の質問では同じプロンプトウィンドウを使い続けてください。セクションが変わったら新しいプロンプトウィンドウを開き、Assistant が各リクエストをまっさらな状態からどう処理するかを確認しましょう。実際の業務では、1つの調査には1つのプロンプトを使い続けることになります。

![Grafana Assistant の起動ボタン](https://grafana.github.io/observability-workshop-breakouts/assets/images/0.1-grafana-assistant-5692ca4ca767d2d550cd20da3d87afc4.png)

時間は20分です。まずは基本の質問を優先し、時間があれば追加課題に挑戦してください。

> **注記**
>
> LLM アシスタントの性質上、結果はユーザーごとに異なったり、これまでの演習で得られた結果と一致しなかったりすることがあります。Assistant が思いどおりに動かない場合は、もう一度プロンプトを送って正しい方向に導いてください。

### 注目してほしいポイント

**Assistant のツールの使い方** - LLM はハルシネーションを起こすことがあるため、Assistant は作業の過程を表示します。tool call を展開して何を考えているのかを確認すれば、その結論を信頼できるかどうか判断できます。

**Assistant のエラー** - Assistant も人間と同じように間違えます。エラーはフィードバックとして Assistant に返され、Assistant は考え直して試行を繰り返します。どうしても結論が出せない場合は、（しばらく試したあとで）たいていその旨を伝えてくれます。

**グラフやログの可視化** - Assistant は Grafana の可視化ライブラリを使い、言葉で*説明する*だけでなく実際に*見せて*くれます。これにより信頼性が高まり、長い文章の羅列も避けられます。

**次のステップの提案ボタン** - ほとんどの場合、Assistant は次に聞くべき質問やアクションを提案してくれます。もちろん、そのままプロンプトを続けても構いません。

- **[3.1. Frontend Observability のデータをクエリする](frontend-o11y.md)**: ここまでで、productcatalog の謎を2通りの方法で解きました。1つは Lab 1 のアプリを使う方法、もう1つは Lab 2 の Knowledge Graph を使ったより短い方法です。3つ目の選択肢があります。どこにも移動せず、ただ聞くだけです。

- **[3.2. Kubernetes Monitoring のデータをクエリする](k8s-monitoring.md)**: Lab 1.3 では、Workloads → Pod → Containers → right-sizing の gauge、とクリックしてたどりました。今度は Assistant に同じ答えを普段の言葉で尋ね、たどる経路がどう変わるかを見てみましょう。

- **[3.3. Span Metrics をクエリする](span-metrics.md)**: Lab 1.2 では、Application Observability でサービスを1つずつ確認しながら、productcatalog の crash loop を追跡しました。ここではもう一度同じことを行いますが、今回は Assistant が主導します。手作業でたどり着いたのと同じ結論に Assistant がたどり着けるかを確認しましょう。
