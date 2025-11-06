# Gemini Demo Setup構想

	•	ルート直下に**00_initialize (for models).md**（またはGoogleドキュメント版）を置く
→ “ここから辿れば全て読める”一本化入口
	•	01_short-facts.md：箇条書きの事実（高頻度更新）
	•	02_ingest-manifest.yaml：読み込み順と除外のヒント（人間規律＋将来フック）
	•	phase-cards/：Baseline/Tx/Targetの1枚カード（各1ファイル、短め）
	•	バイナリやPDFは**_evidence/**に集約し、本文に要点だけ転記（リンクで参照）

```
/AI-Assisted EA Demo (Driveのフォルダ)
├── 00_initialize (for models).md   ← ★まずこれを開かせる
├── 01_short-facts.md
├── 02_ingest-manifest.yaml
├── phase-cards/
│   ├── 00_baseline.md
│   ├── 10_transition-1.md
│   └── 99_target.md
├── 01_working/ef-solutions/20_transition-arch/.../00_overview/README.md
├── 02_adr/...
├── 03_arch-artifacts/backcasting/{nodes.csv,relations.csv}
└── _evidence/ (画像・PDF・ログ等)
```

## 入口ドキュメントの中身（そのままコピペでOK）

00_initialize (for models).md

```
# Initialize (for Models)

## Role
You are an Enterprise Architecture reviewer for a modernization journey (Baseline → Transition → Target).

## Inputs
- Facts: ./01_short-facts.md
- Phase Cards: ./phase-cards/*
- Transition Snapshots (overviews only):
  - ./01_working/ef-solutions/20_transition-arch/**/00_overview/README.md
- ADR Catalog: ./02_adr/
- Backcasting: ./03_arch-artifacts/backcasting/nodes.csv, relations.csv

## Tasks
1) Executive Summary (≤10 bullets)
2) Risk Map (cause→effect chains, 5–7)
3) ADR Recommendations (3, cite sources)
4) NFR Deltas (improve/regress + reasons)
5) Next-Phase Options ({cost, lead-time, risk})

## Notes
- Ignore `_evidence/` unless explicitly linked.
- If any file feels redundant, prefer the Phase Card + short-facts.
```

01_short-facts.md（例）

```
- Baseline: Shared DB + nightly batch; p95 /checkout 800ms.
- Tx-1: API GW + BFF導入、WebAppはblue/green。目標p95 600ms。
- Tx-2: OrderドメインをStranglerで切り出し、outbox経由イベント化。目標450ms。
- Target: Product-aligned platform、stream-aligned + platform teams。
```

02_ingest-manifest.yaml

```
include:
  - "00_initialize (for models).md"
  - "01_short-facts.md"
  - "phase-cards/*.md"
  - "02_adr/**/*.md"
  - "03_arch-artifacts/**/*.{md,csv,json,yaml,yml}"
  - "01_working/ef-solutions/20_transition-arch/**/00_overview/*.md"
exclude:
  - "_evidence/**"
  - "**/*.png"
  - "**/*.pdf"
  - "**/.git/**"
notes:
  - "重い証憑は本文に要点を転記、原本はリンク参照。"
```

## Driveならではの小ワザ

	•	ショートカットを活用：phase-cards から各 00_overview への Driveショートカット を作ると、同一実体を多重配置せずに“入口からの距離”を縮められる。
	•	Googleドキュメント版も用意：00_initialize は .md と Googleドキュメントの二枚看板にすると、プレビュー互換で外部コラボもしやすい（どちらも同内容）。
	•	番号プレフィクスで読み順固定：00_, 01_, 02_ を入口付近だけでも付ける。
	•	1ファイル1論点：長文は分割。Geminiは短い塊の方が要約・参照精度が安定。

## “Rootから丸呑み”運用の落とし穴を避ける

	•	画像・PDFの氾濫：_evidence/に隔離。本文にサマリを貼る（Geminiのトークン節約＆要約ブレ低減）。
	•	入れ子深すぎ問題：入口から3クリック以内でBasline/Tx/Targetの要点に到達できるよう、Phase Card＋ショートカットで中継点を作る。
	•	重複定義：正史（02_adr/, 03_arch-artifacts/principles.md など）を“1か所”に決め、他はリンクで参照。

## まずやる3手

	1.	ルートに上の3ファイル（00/01/02）を作る。
	2.	phase-cards/00_baseline.md と phase-cards/10_transition-1.md を置く（各20行以内）。
	3.	DriveのRootフォルダをGeminiで開く→00_initializeを開かせるルーチンを作る（ブックマークでも良い）。