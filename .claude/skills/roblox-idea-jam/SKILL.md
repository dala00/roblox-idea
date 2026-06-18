---
name: roblox-idea-jam
description: Robloxの新作ゲームのアイデアを大量生成→採点→順位付けし、HTMLレポートにまとめる。idea-generatorを約10並列で走らせ、idea-criticで独立採点し、ideas/<timestamp>/ にHTMLとJSONを出力する。新しいRobloxゲームのアイデア出し・ブレスト・アイデアコンペをしたいときに使う。
---

# Roblox Idea Jam

Robloxの新作ゲームのアイデアを並列生成し、独立審査で採点・順位付けして、見やすいHTMLレポートにまとめるスキル。

引数(任意): テーマ/縛り(例: `/roblox-idea-jam ホラー寄りで` `--trends`(トレンドを強制再検索) `--count=12`)。無指定なら全方位 × 10本。

## 全体の流れ
1. トレンド接地(キャッシュ or web検索)
2. 多様性シードを配って約10本を**並列生成**
3. 重複の統合
4. **並列で独立採点** → 加重合計で順位付け
5. `ideas/<yyyymmddhhiiss>/` に HTML + JSON を出力

---

## Step 1: トレンド接地
`research/` 配下の最新の `roblox-trends-*.md` を探す(Glob `research/roblox-trends-*.md`)。
- ファイルがあり、その先頭の `調査日時:` が**今日から60日以内**なら、**確認も再検索もせず**その md を読んで `trends` 要約として使う。
- **無い／60日より古い／引数に `--trends` がある場合は、すぐに検索せず、まずユーザーに「トレンドを再検索して更新しますか？」と確認する**(AskUserQuestion 等)。
  - **yes の場合のみ** 下記を **WebSearch** で調べて新しい md を保存する:
    - 「Roblox 人気ゲーム ジャンル トレンド 2026」「Roblox top games genres」「Roblox 流行 simulator tycoon roleplay」等
    - 何が伸びているか/飽和しているか、若年層の嗜好、課金の効いているジャンル。
  - **no の場合**: 既存の最新 md があればそれを(古くても)そのまま使う。1件も無ければトレンド接地なしで続行する旨を伝えて進める。
- 検索した(yesの)ときは結果を `research/roblox-trends-<yyyymmdd-hhmmss>.md` に保存する。先頭に必ず:
  ```
  調査日時: YYYY-MM-DD HH:MM:SS
  ```
  続けて「伸びているジャンル / 飽和ジャンル / 若年層の嗜好 / 課金の効くポイント / 差別化の余地」を箇条書きで。
- この `trends` 要約を以降の generator に渡す。

(現在日時は会話冒頭の currentDate / 実時間から判断。タイムスタンプはこの値を使う。)

## Step 2: 並列生成(約10本)
以下の**多様性シード**から count 本(既定10)を選び、各シードを1エージェントに割り当てて **roblox-idea-generator を1メッセージ内で並列起動**する(Agent tool, subagent_type=`roblox-idea-generator`)。同じプロンプトで10並列にすると似た案ばかりになるため、必ずシードを変える。

既定シード(10):
1. 物理ベースのドタバタ(観戦も笑える / chaos)
2. 制作・クラフト・自己表現(build/craft)
3. 協力サバイバル(co-op survival)
4. 非対称対戦(1 vs many / 鬼ごっこ系)
5. 経営・タイクーン・放置成長(tycoon/idle)
6. 探索・謎解き・発見(exploration/mystery)
7. ソーシャルなりきり・たまり場(roleplay/hangout)
8. リズム・タイミング・反射(rhythm/timing)
9. ペット/キャラ育成・収集(collect & raise)
10. レース・パルクール・obby(movement)

各 generator へのプロンプトに必ず含める:
- `seed`: 上記のうち1つ(+ユーザー引数のテーマ縛りがあれば合成)
- `trends`: Step 1 の要約
- `avoid`: (任意)既に他で出ていれば差別化指示

引数でテーマ縛りがある場合は全シードにそれを掛け合わせる。`--count` 指定時は本数を調整(シードが足りなければ既存シードに別の「狙う感情/一捻り」を付けて派生させる)。

各 generator の返答から ```json ブロックを抜き出して配列にまとめる。パースできない物はもう一度該当エージェントだけ起動するか除外。

## Step 3: 重複の統合
集まったアイデアを見比べ、**コアが実質同じもの**はクラスタにまとめて代表1本に統合(タイトルとpitchで判断)。統合した場合は良い要素を取り込む。最終的にユニークな候補リストにする。

## Step 4: 並列採点 → 順位付け
候補1本ごとに **roblox-idea-critic を並列起動**(subagent_type=`roblox-idea-critic`)。1メッセージで全件ファンアウトする。各 critic には**アイデアのJSONを丸ごと**渡す(生成役の自己採点を避けるため、generatorの出力をそのまま採点対象として渡すだけ)。
- 返ってきた ```json から scores / weighted_total / effort / verdict / biggest_risk / improvement を取得。
- weighted_total が崩れていたら自分で再計算(重み: originality1.4, vibe0.8, fun1.4, accessibility1.0, monetization1.2, social0.9, solo0.8, feasibility1.2, virality1.3 / 合計10.0・満点100)。**9観点**であることに注意。
- weighted_total 降順で順位付け。同点は originality+fun+virality の合計で割る。

## Step 5: 出力
`ideas/<yyyymmddhhiiss>/` を作成(タイムスタンプは実行時刻。例 `ideas/20260618143000/`)。
- `ideas.json`: 全候補の生成データ + 採点 + 順位の配列(再採点・再利用できる生データ)。
- `report.html`: 単一HTMLの見やすいレポート。CSSはインライン、外部依存なし。次を含める:
  - ヘッダ: 生成日時、テーマ/引数、参照したトレンドmdのパスと調査日時、件数。
  - **ランキング表**: 順位 / タイトル / tagline / weighted_total / effort / 9観点スコア(virality含む)。
  - **カード**: 上位から各アイデアの pitch・surprise・core_loop_30s・progression・social_hook・solo_ok・clip_appeal・monetization・roblox_feasibility と、critic の verdict・biggest_risk・improvement、9観点スコアのバー表示。
  - スコアバーは `<div>` 幅 % で表現(JS不要)。日本語表示。読みやすい配色。
- 出力後、レポートの**絶対パス**と上位3案の要約を会話で報告する。

## メモ
- generator/critic は別エージェントなので自己採点バイアスは避けられている。さらに厳密にしたい場合は critic を1案あたり2回起動して平均してもよい(既定は1回)。
- `ideas/` は .gitignore 済み(出力は履歴に乗せない)。`research/` のトレンドmdはキャッシュとして残す。
