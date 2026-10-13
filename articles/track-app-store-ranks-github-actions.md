---
title: "GitHub Actions で App Store の順位とレビュー数を毎日 0 円で記録する"
emoji: "📈"
type: "tech"
topics: ["appstore", "ios", "githubactions", "python", "aso"]
published: true
published_at: 2026-10-28 09:00
---

:::message
この記事の筆者は、iOS アプリを個人で開発している Lumi Studio（台湾）の開発者です。紹介する仕組みは自作アプリの計測用に作ったもので、末尾で一言だけ触れます。本文は Apple の公開 API と GitHub Actions だけで再現できる内容です。
:::

## 作ったもの

53本の iOS アプリを、それぞれ50言語で出しています。ストアフロントごとの評価数、検索順位、カテゴリチャートの順位を手作業で見て回るのは現実的ではないので、GitHub Actions で毎日自動計測し、結果を1つの Issue に書き込む仕組みを作りました。実装は公開リポジトリ `alice51849/ios-app-guide` の `.github/scripts/growth_scoreboard.py` にあります。

- Apple の公開エンドポイントを読むだけで、App Store Connect の認証情報は使いません
- Python の標準ライブラリだけで動き、サーバーは不要です
- 進捗と結果は `actions/cache` に JSON で保存し、出力は Issue の本文に書きます

タイトルの「0 円」は、標準ランナーが無料の公開リポジトリで動かしているという意味です。プライベートリポジトリでは無料枠を消費するので、各自の契約を確認してください。「レビュー数」は、ここでは評価の件数（`userRatingCount`）を指します。

:::message alert
得られる順位は、公開 API を呼んだ時点の観測値です。Apple の公式データではなく、App Store の画面で実際に見える順位と一致する保証もありません。
:::

## 使う3つの公開エンドポイント

いずれも `https://itunes.apple.com` 以下です。

| 目的 | パス | 得られるもの |
|---|---|---|
| 評価数 | `/lookup` | 店面ごとの評価件数と平均 |
| 検索順位 | `/search` | `limit=200` で、結果の並び＝順位 |
| チャート | `/{cc}/rss/topfreeapplications/limit={n}/genre={id}/json` | 上位最大100件 |

Lookup は `id` をカンマ区切りで複数渡せるので、100本ずつまとめて店面ごとに1リクエストで済みます。ここで得たジャンル ID と価格から、どのカテゴリの無料／有料チャートを見るかを決めます。チャートは古い形式の RSS フィードで、エントリが1件だけのときに配列ではなくオブジェクトで返る場合にも備えています。

検索順位は、結果の配列で自分のアプリの `trackId` が何番目に出てくるかを数えるだけです。チャートも並びの位置がそのまま順位になります。ID を取れなかった行にも `None` を入れて位置を保ち、1件の不備で後ろの順位がずれないようにしています。検索では1位のアプリ名と評価数も記録し、「1位の評価数が少ないから狙えそうだ」という見立てに使います。

```python
def search_rank(term, cc, my_ids):     # 圏外は None
    q = urllib.parse.urlencode({"term": term, "country": cc,
                                "entity": "software", "limit": 200})
    ranks = {}
    for pos, r in enumerate(get_json(f"{BASE}/search?{q}")["results"], 1):
        ranks.setdefault(str(r["trackId"]), pos)
    return {i: ranks.get(i) for i in my_ids}
```

## 間隔を空け、止められたら止まる

外部 API を毎日叩くので、礼儀を守る部分を最初に作りました。簡略化した抜粋です。

```python
class RateLimited(RuntimeError): ...   # 403/429: 今回は打ち切る

def get_json(url):
    time.sleep(3.1)                    # 毎回、間隔を空ける
    req = urllib.request.Request(url, headers={"User-Agent": USER_AGENT})
    try:
        with urllib.request.urlopen(req, timeout=30) as res:
            return json.loads(res.read())
    except urllib.error.HTTPError as e:
        if e.code in (403, 429):
            raise RateLimited(f"HTTP {e.code}") from e
        raise
```

実装では前回リクエストからの経過時間を見て待ち、408 や 5xx のような一時的エラーだけは最大3回までリトライします。403 と 429 はリトライせず、例外にして呼び出し側へ返します。待てば直るエラーと、これ以上叩くべきでないエラーを分けるためです。User-Agent には、連絡先が分かるプロジェクト名を入れています。

## 毎日の予算で回すローテーション

計測対象は「アプリ × ロケール × キーワード」の組です。出品用のキーワードは1ロケールあたり3つまでなので、それだけなら理論上の最大は 53 × 50 × 3 = 7,950 組です（狙いのキーワードが別に加わり、実数は登録状況次第）。1日 600 件の予算では一周に13日ちょっとかかるため、全部を毎日は回さず、リング状のリストをカーソルで進める方式にしました。

```python
def take_rotation(items, cursor, budget):
    start = cursor % len(items)
    n = min(budget, len(items))
    return [items[(start + i) % len(items)] for i in range(n)], (start + n) % len(items)

picked, next_cursor = take_rotation(tasks, state["cursor"], budget)
start, done = state["cursor"], 0
try:
    for t in picked:
        ranks = search_rank(t["keyword"], t["country"], my_ids)
        done += 1
        state["search"][task_id(t)] = {"rank": ranks[t["app_id"]]}
    state["cursor"] = next_cursor
except RateLimited:
    state["cursor"] = (start + done) % len(tasks)   # 止められた所から再開
```

見直し済みの「狙いのキーワード」を先頭に置き、その後は各アプリの第1キーワードを全ロケールで一巡してから第2キーワードへ進みます。タスク一覧のハッシュが前回と変わったとき（アプリやキーワードの追加）は、カーソルを0に戻して新しい項目を先に測ります。

Apple から 403 や 429 が返ったら、その日はそこで諦めます。`done` は処理を終えた分だけ数えるので、止められたリクエストが次回の先頭になります。例外は `run()` の中で受け、正常終了と同じ流れで状態の保存とレポートまで行います。レポートには途中停止の旨を1行出します。状態ファイルは一時ファイルに書いてから `os.replace` で差し替えるので、途中で落ちても壊れた JSON は残りません。

待ち時間だけで、既定値（検索 600、チャート 450）なら 1,050 × 3.1 秒、約54分かかります。

## 状態は actions/cache、レポートは Issue

Actions のランナーは毎回まっさらなので、状態の置き場が要ります。キャッシュは同じキーに上書きできないため、実行ごとに `run_id` を含む新しいキーで保存し、`restore-keys` の前方一致で直近のものを復元します。状態には評価数の履歴も入っていて、前回比はここから出します。キャッシュは永続ストレージではありません。長く使われないと GitHub 側で削除されることがあり、消えれば履歴は最初からです（保持条件は GitHub のドキュメントで確認してください）。念のため、全測定結果を artifact にも90日間残していますが、自動復元には使っていません。

結果は専用の Issue 本文を毎日上書きします。同じタイトルのオープンな Issue を探して更新し、なければ作ります。Issue 一覧は PR も返すので、`pull_request` キーの有無で除きます。本文の長さには上限があるため、60,000 文字で切り詰めています。中身は、評価数の合計と前回比、検索で1位／10位以内に入った組の数、30位以内なのに1位の評価数が少ない「あと一歩」の組、カテゴリチャートの順位です。

毎日必ず追いたい少数の対象は、ローテーションとは別枠で毎日測っています。計測の前にはネットワーク不要のテストを走らせています。

## ワークフローの骨子

setup-python などは省略しています。

```yaml
on:
  schedule: [{ cron: "41 2 * * *" }]   # UTC
permissions: { contents: read, issues: write }
concurrency: { group: growth-scoreboard, cancel-in-progress: false }
jobs:
  measure:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache/restore@v4
        with:
          path: .growth-scoreboard
          key: growth-scoreboard-v1-${{ github.run_id }}-${{ github.run_attempt }}
          restore-keys: growth-scoreboard-v1-
      - run: python3 .github/scripts/growth_scoreboard.py --state .growth-scoreboard/state.json --report .growth-scoreboard/report.md --issue
        env: { GITHUB_TOKEN: "${{ github.token }}" }
      - if: always() && hashFiles('.growth-scoreboard/state.json') != ''
        uses: actions/cache/save@v4
        with:
          path: .growth-scoreboard
          key: growth-scoreboard-v1-${{ github.run_id }}-${{ github.run_attempt }}
```

権限は最小にし、`concurrency` で二重実行を防ぎ、保存ステップを `always()` にして計測が失敗しても進捗を残します。

## 限界と注意

- **Search API の制限**：続けて叩くと 403 や 429 で止められることがあります。閾値は Apple の都合で変わりうるので、間隔を空け、止まったら止まる作りにしています。
- **検索は200位まで**：圏外は「200+」としか分かりません。順位は日や時間帯でも動くので、目安として扱ってください。
- **チャートは最大100件**：`limit` には 200 を渡していますが、返ってくるのは最大100件でした。古い形式のフィードで、いつまで使えるかの保証もありません。
- **更新頻度**：ローテーションなので、組数が最大のとき、1つの組が更新されるのは十数日に1回です。
- **利用条件**：Apple の API の利用条件は、各自で確認してください。

## 最後に

筆者はこの仕組みを、自作の53本の iOS アプリの計測に使っています。関連資料は次のガイドサイトにまとめています。

https://open.cait518.cc/ios-app-guide/
