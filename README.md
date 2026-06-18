# Roblox Game Idea Jam

Robloxの新作ゲームのアイデアを出すための環境。

## 使い方
Claude Code で次のスキルを実行する:

```
/roblox-idea-jam
```

引数の例:
- `/roblox-idea-jam ホラー寄りで` … テーマ縛りを掛ける
- `/roblox-idea-jam --count=12` … 本数を増やす
- `/roblox-idea-jam --trends` … 最新トレンドを強制的に再検索する

## 何が起きるか
1. 最新のRobloxトレンドを接地(`research/` のキャッシュが7日以内ならそれを使用、古ければweb検索して保存)
2. 多様性シードを配って約10本のアイデアを**並列生成**(`roblox-idea-generator`)
3. 重複を統合
4. 別人格の審査員が**独立採点**(`roblox-idea-critic`)し、加重合計で順位付け
5. `ideas/<yyyymmddhhiiss>/` に `report.html` と `ideas.json` を出力

## 構成
- `.claude/agents/roblox-idea-generator.md` … アイデア発案サブエージェント
- `.claude/agents/roblox-idea-critic.md` … 採点サブエージェント
- `.claude/skills/roblox-idea-jam/SKILL.md` … 並列生成→採点→レポート化のスキル
- `research/` … トレンド調査のキャッシュ(調査日時付きmd)
- `ideas/` … 出力(.gitignore済み)

## 採点観点(各1〜10 / 加重合計100点満点)
オリジナリティ・驚き(×1.5) / 雰囲気(×1.0) / 実際の面白さ(×1.5) / 間口(×1.2) /
マネタイズ(×1.3) / ソーシャル・招待性(×1.2) / ソロ・少人数耐性(×1.0) / Roblox個人開発の実現性(×1.3)
