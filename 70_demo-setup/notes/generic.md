# Setup Memo (Generic)

	•	60_for-models/ を“正規の入口（canonical entrypoint）”にする。
	•	ルート README.md は極小にして、for-models への矢印だけ置く。
	•	人向けは分離して 70_for-humans/（または docs/）にまとめる。

```
root/
├── 60_for-models/
│   ├── 00_initialize.md        # ★AIが最初に読む（常にここを渡す）
│   ├── 01_short-facts.md       # 事実の要約（更新頻度高）
│   ├── 02_ingest-manifest.yaml # 取り込み順序と除外（擬似 .gitignore 的）
│   ├── 03_index.json           # 重要ドキュメントの機械可読インデックス
│   └── phase-cards/            # 各段の“1枚カード”
│       ├── 00_baseline.md
│       ├── 10_transition-1.md
│       └── 99_target.md
└── 70_for-humans/
    └── README.md               # 人間向けの道案内（任意）
```

ルート README.md（最小）

```
# AI-Assisted EA Demo
Primary entry for GenAI: `./60_for-models/00_initialize.md`  
Human-oriented overview: `./70_for-humans/README.md`
```

## 取り込み（ingest）を安定させる3点

### 1) 入口の固定（00_initialize.md）

	•	役割/前提/入出力/期待する出力セクションを固定文面で。
	•	依存ファイルは相対パスで列挙（short-facts、phase-card、ADRカタログ等）。

例：

```
# Initialize (for Models)

## Role
You are an EA reviewer...

## Inputs
- Facts: ./01_short-facts.md
- Phase Cards: ./phase-cards/*
- ADR Catalog: ../02_adr/
- Transition Snapshots: ../01_working/ef-solutions/20_transition-arch/**/00_overview/README.md
- Backcasting: ../03_arch-artifacts/backcasting/{nodes.csv,relations.csv}

## Tasks
1) Executive Summary (≤10 bullets)
...
```

### 2) 取り込み順と除外（02_ingest-manifest.yaml）

“rootからがさっと読み込む”運用でも、順序と除外があると安定します。

```
include:
  - "60_for-models/00_initialize.md"
  - "60_for-models/01_short-facts.md"
  - "60_for-models/phase-cards/*.md"
  - "03_arch-artifacts/**/*.{md,csv,yml,yaml,json}"
  - "02_adr/**/*.md"
  - "01_working/ef-solutions/20_transition-arch/**/00_overview/*.md"
exclude:
  - "**/*.png"
  - "**/*.pdf"
  - "04_evidence/**"
  - "**/node_modules/**"
  - "**/.git/**"
notes:
  - "Images/PDFは参照リンクのみ。本文に重要事実を転記すること。"
```

ツール側に“manifest理解”がなくても、人間編集の規律として効くし、将来の自動化フックにも使えます。

### 3) 重要物の機械可読インデックス（03_index.json）

モデルに“どれが核か”を明示。

```
{
  "facts": "60_for-models/01_short-facts.md",
  "phases": [
    {"id":"00_baseline","card":"60_for-models/phase-cards/00_baseline.md"},
    {"id":"10_transition-1","card":"60_for-models/phase-cards/10_transition-1.md"},
    {"id":"99_target","card":"60_for-models/phase-cards/99_target.md"}
  ],
  "adr_catalog": "02_adr/",
  "backcasting": {
    "nodes": "03_arch-artifacts/backcasting/nodes.csv",
    "relations": "03_arch-artifacts/backcasting/relations.csv"
  }
}
```

## 人向けの分離（70_for-humans/）
	•	ここは解説・図多めでOK。モデルは基本無視でよい。
	•	将来Docs化するなら docs/ でも良いけど、モデル入口と混ぜないのがコツ。

70_for-humans/README.md 例

```
# How to read this repo (for humans)
- Start: modernization intent, constraints, evaluation axes
- Then see: phase cards for Baseline/Tx/Target
- Model entry is separate: `../60_for-models/00_initialize.md`
```

## 運用の細ルール（効きどころだけ）
	•	数値プレフィクスで読み順を制御（00_, 01_, …）。
	•	1ファイル1論点（モデルのトークン効率が上がる）。
	•	重い証憑はリンクのみ（本文に要点を転記）。
	•	更新印：各ファイル先頭に Last-Updated: YYYY-MM-DD と Trust: draft|reviewed|final を置くと、モデルの判断が安定。

例（phase cardの頭）

```
# 10_transition-1 (Phase Card)
Last-Updated: 2025-11-06
Trust: reviewed
Intent: Decouple web from monolith via BFF + API GW
NFR: p95 /checkout 600ms target
Key Changes: ...
Risks: ...
```

