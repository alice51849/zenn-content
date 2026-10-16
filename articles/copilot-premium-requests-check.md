---
title: "GitHub Copilot の残り（AIクレジット・プレミアムリクエスト）を確認する方法"
emoji: "🧮"
type: "tech"
topics: ["githubcopilot", "github", "vscode", "ai"]
published: true
published_at: 2026-10-25 09:00
---

:::message
この記事の最後で紹介している iOS アプリ「UsageNow」は筆者が開発したものです。前半では、無料で使える公式の方法とオープンソースのツールだけを紹介しています。
:::

「プレミアムリクエストの残りはどこで見るんだっけ」と思って調べると、2026年6月の課金変更の前後で情報が混ざっていて分かりにくくなっています。今のプランで何を見ればよいのか、どこで確認できるのか、いつリセットされるのかを、GitHub の公式ドキュメントで確認できた範囲で整理しました。

:::message alert
この記事は2026年10月16日時点の docs.github.com の内容をもとにしています。Copilot の課金と上限は変更が続いているので、最終的には公式ドキュメントと自分のアカウントの表示で確認してください。
:::

## 先に結論

| やりたいこと | 方法 |
|---|---|
| エディタで今月の使用率とリセット日を見たい | VS Code のステータスバーの Copilot アイコン（他の IDE もあり） |
| ブラウザで内訳まで見たい | github.com の Billing → **AI usage** |
| ざっくり今月の消費だけ見たい | Copilot settings の **Usage** |
| 使いすぎを防ぎたい | Billing の **Budgets and alerts** で予算とアラートを設定 |
| Copilot CLI のセッションでの使用量を見たい | Copilot CLI の `/usage` |
| ターミナルやエディタを離れているときに、他サービスとまとめて見たい | スマートフォンのウィジェット（最後に紹介） |

リセットは**毎月1日 00:00:00 UTC**、日本時間では**毎月1日の午前9時**です。

## 前提：2026年6月から「プレミアムリクエスト」は「AIクレジット」に

GitHub は2026年6月1日に、Copilot の全プランを使用量ベースの課金に移行しました。公式ブログと docs によると、ポイントは次のとおりです。

- それまでのプレミアムリクエストに代わって、**GitHub AI Credits（AIクレジット）**の月間枠がプランに含まれるようになった。
- 消費量は、入力・出力・キャッシュのトークン数と、モデルごとの料金から計算される。
- **コード補完と Next Edit Suggestions は AI クレジットを消費しない。**有料プランでは無制限で、Copilot Free ではコード補完が月2,000回まで。
- 使い切ったときに安いモデルへ切り替えて使い続ける仕組み（フォールバック）はなくなった。

一方、**年額プランの Copilot Pro／Pro+ の人は、プランの期限が切れるまで従来のプレミアムリクエスト方式のまま**です。月額プランの人は6月1日に自動で移行しています。

つまり、確認すべきものは人によって違います。

- 月額プラン、Business／Enterprise → **AIクレジット**
- 年額の Pro／Pro+ を継続中 → **プレミアムリクエスト**

どちらでも、以下の確認場所はほぼ共通です。

## 方法1：エディタで確認する（公式・無料）

いちばん手軽なのは、エディタの Copilot アイコンです。docs に載っている手順は次のとおりです。

| エディタ | 手順 |
|---|---|
| Visual Studio Code | ステータスバーの Copilot アイコンをクリック |
| Visual Studio | 右上の Copilot アイコン → **Copilot Consumptions** |
| JetBrains IDE | ステータスバーの Copilot アイコン → **View quota usage** |
| Xcode | メニューバーの Copilot アイコンをクリック |
| Eclipse | ウィンドウ下部のステータスバーの Copilot アイコンをクリック |

VS Code の場合、表示されるのは次の内容です（VS Code 公式 FAQ より）。

- 今月使ったインライン補完の割合（%）
- 今月使った AI クレジットの割合（%）
- 枠がリセットされる日

AI クレジットだけを使い切った場合でもインライン補完は使えますし、その逆も同じです。なお、**月間の AI クレジットが残っていても、一時的なレート制限にかかることはあります**。その場合は数分待ってから再試行するよう案内されています。「残っているのに止まった」ときは、こちらを疑ってください。

## 方法2：github.com の AI usage で内訳を見る（公式・無料）

個人プランの場合、ブラウザで次の手順で開きます。

1. https://github.com/settings/billing を開く
2. サイドバーの **AI usage** をクリック

ここでは次の情報が見られます。

- プランに含まれるクレジットのうち使った量
- 含まれる分を超えた追加利用
- 使ったモデルのグラフ
- モデルごとのクレジットと金額の内訳

「どのモデルでクレジットを使っているか」が分かるので、思ったより減りが速いときの原因探しに向いています。なお、この AI usage のページは個人プラン（Copilot Individual）専用です。

使用量のレポートを CSV などでダウンロードしたい場合は、Billing の docs にある「Downloading usage reports」の手順を参照してください。

### 年額プランでプレミアムリクエストのままの人

従来方式の人向けの docs では、確認場所が次のように案内されています。

- https://github.com/settings/billing の **Overview** で、**Metered usage** の欄の Copilot アイコンをクリックすると Copilot の使用量だけを表示
- サイドバーの **Premium request analytics** で、フィルタや **Group by**、**Timeframe** を使って集計

プレミアムリクエストのカウンタも、毎月1日 00:00:00 UTC にリセットされます。

## 方法3：Copilot settings の Usage（公式・無料）

内訳までは要らない、今月どれだけ使ったかだけ知りたい、というときはこちらが手軽です。

1. github.com の右上のプロフィール画像をクリック
2. **Copilot settings** をクリック
3. **Usage** の欄で、今の請求期間に使った AI クレジットを確認

Business／Enterprise の人もここで確認できます。管理者がユーザーごとの予算を設定している場合は、「450 / 1,000 AI credits used」のように予算に対する使用量が表示されます。クレジットを使い切ったときは、github.com 上のバナーから管理者に予算の追加を依頼できます。

## 方法4：予算とアラートで使いすぎを防ぐ（公式・無料で設定可能）

個人プランでは、含まれる分を超えた追加利用に予算（米ドル建て）を設定できます。予算を設定しなければ、含まれる分を使い切った時点で、翌月のリセットを待つかプランを上げるかになります。

設定は https://github.com/settings/billing の **Budgets and alerts** → **New budget** から行います。

- **Budget Type** で、Product-level、SKU-level、Bundled AI credits のいずれかの予算タイプを選ぶ
- **Receive budget threshold alerts** を選ぶと、予算の **75%・90%・100%** でメールと GitHub 上のバナーで通知される
- 表示される場合は **Stop usage when budget limit is reached** を選ぶと、予算に達した時点で利用を止められる

「気づいたら追加利用がかさんでいた」を防ぐために、追加利用を有効にするなら一緒に設定しておくのがおすすめです。

## 方法5：Copilot CLI の `/usage` と ccusage（CLI を使う人向け）

ターミナルで Copilot CLI を使っている場合は、セッション内で次のコマンドが使えます。

```text
/usage
```

docs では、そのセッションで消費したクレジット、セッションの経過時間、編集した行数の合計、モデルごとのトークン使用量を表示すると説明されています。あくまで**そのセッションの**数字なので、月間の残りは方法1〜3で確認してください。

また、OSS の [ccusage](https://github.com/ryoppippi/ccusage) は Copilot CLI の手元のログにも対応しています。

```bash
npx ccusage@latest copilot daily
```

日別の使用量を振り返るのに便利ですが、手元のマシンの Copilot CLI のログだけが対象で、エディタや github.com での利用、公式の月間の残り（%）は含まれません。

## 比較

| 方法 | わかること | 公式の残り | 見る場所 | 料金 |
|---|---|---|---|---|
| エディタの Copilot アイコン | 今月の使用率、リセット日 | ○ | エディタ | 無料 |
| Billing → AI usage | 使用量、追加利用、モデル別の内訳 | ○ | ブラウザ | 無料 |
| Copilot settings → Usage | 今の請求期間の使用量 | ○ | ブラウザ | 無料 |
| Budgets and alerts | 追加利用の予算と通知 | ― | ブラウザ・メール | 設定は無料 |
| Copilot CLI `/usage` | そのセッションの使用状況 | ×（セッション単位） | ターミナル | 無料 |
| ccusage | Copilot CLI の日別の集計 | ×（集計のみ） | ターミナル | 無料（OSS） |

## 使い分け

- エディタで作業していて、たまに確認できればいい → ステータスバーの Copilot アイコン。
- 減りが速い理由を知りたい → Billing の AI usage でモデル別の内訳を見る。
- 追加利用を有効にしている → Budgets and alerts でアラートを設定しておく。
- 月末が近いときは「何日の何時にリセットされるか」が大事です。日本時間なら**毎月1日の午前9時**と覚えておくと便利です。
- エディタを離れているとき（移動中、会議中、月末に残りだけ確認したいとき）や、Claude・Codex・Gemini も併用していて、それぞれの残りとリセット時刻をまとめて見たい → スマートフォンで見られるものがあると便利です。このケースのために筆者が作ったのが次のアプリです。

## 最後に：筆者のアプリ UsageNow について

※ここからは筆者が開発したアプリの紹介です。

UsageNow は、各 AI プランの残りとリセット時刻を iPhone のホーム画面に表示するアプリです。

- **対応する上限**：GitHub Copilot のプレミアムリクエスト・チャット・コード補完、Claude の5時間セッション上限と週間上限（モデル別の週間上限も含む）、ChatGPT Codex の5時間枠と週間枠、Gemini のモデルごとの1日あたりのリクエスト上限
- **カウントダウン**：リセットまでのカウントダウンと、正確なリセット時刻を地域の表記で表示
- **アラート**：90 や 98 など好きなパーセンテージを入力して、上限に届く前に通知。上限ごと・サイクルごとに1回だけ届きます。上限が戻った瞬間を知らせるリセットアラートも
- **複数アカウント**：仕事用と個人用などを並べて表示し、リセットが近い順や使用量の多い順などで並べ替え
- **ウィジェット**：ホーム画面・ロック画面の全サイズ。ウィジェット上で全アカウントをめくれて、タップで更新。Apple Watch のコンプリケーションにも対応
- **対応端末**：iPhone、iPad、Apple Watch

サインインは各サービス公式のログインページで行い、サインイン情報はデバイスのキーチェーンに保存されます。UsageNow のサーバはなく、アプリが通信するのは各サービスだけで、アカウント登録もトラッキングもありません。

https://apps.apple.com/jp/app/id6818607603?pt=118326163&ct=bm_usagenow_jp_zenn&mt=8

UsageNow は独立した個人開発アプリで、Anthropic、OpenAI、GitHub、Microsoft、Google とは提携していません。Claude、ChatGPT、Codex、GitHub Copilot、Gemini は各所有者の商標です。

エディタの前にいる間は、ステータスバーの Copilot アイコンで十分です。手元を離れたときにだけ必要になる道具として、選択肢の一つにしてもらえたらうれしいです。

## 参考リンク

- [GitHub Copilot is moving to usage-based billing（GitHub Blog）](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)
- [Usage-based billing for individuals（GitHub Docs）](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)
- [Monitoring your AI usage（GitHub Docs）](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/monitor-ai-usage)
- [Monitoring premium requests（従来方式・GitHub Docs）](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/monitor-premium-requests)
- [Setting up budgets（GitHub Docs）](https://docs.github.com/en/billing/how-tos/set-up-budgets)
- [Using GitHub Copilot CLI（GitHub Docs）](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [GitHub Copilot FAQ（VS Code Docs）](https://code.visualstudio.com/docs/agents/agent-troubleshooting/faq)
- [ccusage（GitHub）](https://github.com/ryoppippi/ccusage)
