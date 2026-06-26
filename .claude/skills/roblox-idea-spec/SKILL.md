---
name: roblox-idea-spec
description: Robloxゲームのアイデア1本を、設計原則(game-design-principles.md)に沿って実装着手できる仕様書に落とし込む。spec-writerで起草→spec-reviewerで独立レビュー→指摘を反映して specs/<slug>/spec.md を出力する。idea-jamで出た案や思いついた案を「作れる仕様」にしたいときに使う。
---

# Roblox Idea → Spec

選んだアイデア1本を、個人開発で実装着手できる**具体的な仕様書(Markdown)**に落とし込むスキル。設計原則に接地し、起草役とレビュー役を分離して品質を担保する。

引数(任意): 対象アイデアの指定（例: `/roblox-idea-spec カラー街` `/roblox-idea-spec 1`(最新jamのrank) `/roblox-idea-spec` (無指定なら最新ideas.jsonから選ぶ)）。アイデアのJSONや自由記述を直接貼ってもよい。

## 全体の流れ
0. 設計原則の接地
1. 対象アイデアの解決
2. **spec-writer** で起草
3. **spec-reviewer** で独立レビュー
4. 指摘を反映して最終化
5. `specs/<slug>/spec.md` に出力

---

## Step 0: 設計原則の接地
`.claude/skills/_shared/game-design-principles.md` を読む（不変の設計原則の単一の正）。要点（特に末尾「仕様化で各観点を埋めるチェックリスト」）を以降の writer / reviewer に渡す。

## Step 1: 対象アイデアの解決
次の優先で対象1本を決める：
- プロンプトにアイデアのJSON/自由記述が**直接ある**ならそれを使う。
- 引数にタイトルやrankがあれば、最新の `ideas/<最新timestamp>/ideas.json`(Globで最新フォルダ)から該当案を取る。
- **無指定なら**最新 `ideas.json` を読み、候補一覧（rank・タイトル・weighted_total）を提示して**どの案を仕様化するかユーザーに確認**(AskUserQuestion 等。rank1を推奨として提示)。ideas.json が無ければ、ユーザーにアイデアの記述を求める。
- 対象が決まったら、その生成データ（pitch/surprise/core_loop/progression/monetization/first_session など）を writer に渡せる形に整える。

## Step 2: 起草（spec-writer）
**roblox-spec-writer** を起動(Agent tool, subagent_type=`roblox-spec-writer`)。プロンプトに渡す：
- `idea`: Step 1 の対象アイデア(JSON/要約まるごと)
- `design_principles`: Step 0 の要点
返ってきた Markdown を draft とする。

## Step 3: 独立レビュー（spec-reviewer）
**roblox-spec-reviewer** を起動(subagent_type=`roblox-spec-reviewer`)。プロンプトに渡す：
- `spec`: Step 2 の draft 全文
- `idea`: 元アイデア
- `design_principles`: Step 0 の要点
返ってきた ```json から findings / missing_principles / biggest_risks / mvp_feedback / readiness を取得。
（起草役と採点役の分離と同じく、writerとreviewerは別人格でバイアスを避ける。）

## Step 4: 最終化
draft に reviewer の指摘を**自分(オーケストレーター)で反映**して最終版にする：
- severity 高・中の findings は該当節を fix に沿って具体化/修正。
- missing_principles があれば該当節を補う。
- 反映しきれない/要ユーザー判断の点は「12. オープンな論点」に残す。
- 末尾に **「## 付録: レビュー反映ログ」** を足し、(a)反映した主な指摘 (b)残したオープン論点 (c)readiness を簡潔に記す。
（reviewer の readiness が「低」かつ穴が大きい場合は、指摘を渡して spec-writer を一度だけ再起動して書き直してもよい。既定はオーケストレーターによる反映。）

## Step 5: 出力
`specs/<slug>/` を作成し `spec.md` を書き出す。
- `<slug>` は案のタイトルから分かりやすい英数字/かな（重複しそうなら末尾に実行時刻 yyyymmddhhiiss）。
- 仕様書は**実装用の作業ドキュメント**なので Markdown 単一ファイルで出す（idea-jam の HTML レポートと違い、編集・実装に使う前提）。
- 出力後、**絶対パス**と、要約（コアループ1行 / MVPの中身 / 最大リスク1つ / readiness）を会話で報告する。

## メモ
- `specs/` は **.gitignore済み**（`ideas/` と同様、出力は履歴に乗せない）。
- 実装に進むときは `roblox-gamedev:*` スキル群（rojo-studio-workflow / world-codegen / datastore / monetization / input-mobile 等）が実装ノウハウの参考になる。仕様の「8.実現性」と突き合わせる。
- さらに厳密にしたい場合は reviewer を2回起動して指摘を統合してもよい(既定は1回)。
