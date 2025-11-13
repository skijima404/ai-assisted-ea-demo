Let your GenAI translate it for you. 🌐🤖

---

# Human Guide to the AI-Assisted EA Demo Repository

このフォルダは、「人間が読むための」説明ドキュメントをまとめたエリアです。  
リポジトリのその他部分は、GenAI が構造的に読みやすいよう最適化されています。

---

## 🎯 このリポジトリの目的

本リポジトリは、  
**AI-Assisted Architecture Governance（AI支援アーキテクチャガバナンス）** を実現するための、最小限・再現可能なデモ環境です。
TOGAF ADMライフサイクルの初期からのアーティファクトの生成の過程をトレースし、最後のガバナンス部分の仕組みを構築することで、レビュー段階のアーキテクチャの透明性とGenAIへのContextの質を担保しています。

- アーキテクチャレビューの自動化  
- 不可逆リスクの検知  
- Ops / Sec / EA のインバリアント管理  
- Baseline → Hotspot → Diff レビューのワークフロー  
- GenAI に渡すコンテキストの最適設計

などを **実際のプロジェクトに持ち込める形** でまとめています。

---

## 📁 リポジトリ構造（人間向けのガイド）

```
ai-assisted-ea-demo/
│
├── 01_working/                    # 作業領域（TOGAF Phase A-F の途中成果物）
│   ├── bd_arch-design/           # TOGAF ADM B-D (現状把握とあるべきアーキテクチャ設計)
│   │   ├── 10_raw-notes/         # 生の観察メモ、インタビュー記録
│   │   └── 20_analysis/          # Raw notesからの分析結果
│   │
│   ├── ef_solutions/             # TOGAF ADM E-F (ソリューション選定、移行計画)
│   │   ├── 10_raw-notes/         # ソリューション検討の生メモ
│   │   ├── 20_analysis/          # ソリューション分析結果
│   │   └── 60_transition-draft/  # Transition Architecture のドラフト検討
│   │       └── pattern-1/        # Pattern 1（段階的マイクロサービス移行）の詳細
│   │
│   └── gh_governance/            # Governance & Review Foundation
│       ├── 10_raw-notes/         # EA失敗パターン、レビューの目的などの考察
│       └── 70_review-foundation/ # レビュー基盤の設計ドキュメント
│
├── 02_adr/                        # Architecture Decision Records（設計判断の記録）
│
├── 03_arch-artifacts/            # 正式なアーキテクチャ成果物（Phase D-F 完成版）
│   ├── arch-vision.md            # アーキテクチャビジョン
│   ├── kpi.md                    # KPI定義
│   │
│   ├── architecture-operating-manual/  # EA運用マニュアル
│   │   ├── process-from-vision-to-implementation.md  # Vision→実装までのプロセス
│   │   └── backcasting-map/      # 失敗予防のためのBackcasting Map
│   │
│   ├── baseline/                 # Baseline Architecture（現行システム）
│   │   ├── container-diagram.md  # C4モデル コンテナ図
│   │   └── functional-map.md     # 機能マップ
│   │
│   ├── principles/               # Architecture Principles
│   │
│   ├── target/                   # Target Architecture（目指す姿）
│   │   └── container-diagram.md  # C4モデル コンテナ図
│   │
│   └── transition/               # Transition Architecture（移行計画）
│
├── 10_standards/                 # 組織標準・ポリシー
│
├── 70_demo/                      # デモ用サンプル・セットアップ資料
│   ├── sample/                   # Backcasting Review のサンプル出力
│   └── setup-notes/              # GenAI セットアップメモ
│
├── for-humans/                   # 人間向け説明ドキュメント（このファイル）
│   └── README.md
│
└── templates/                    # 各種テンプレート
```

### フォルダの役割と関係性

#### 📝 作業プロセスの流れ
```
01_working/*/10_raw-notes/        (観察・インタビュー)
    ↓
01_working/*/20_analysis/         (分析・整理)
    ↓
02_adr/                           (設計判断の記録)
    ↓
03_arch-artifacts/principles/     (原則の確立)
    ↓
03_arch-artifacts/baseline/       (現状把握)
    ↓
03_arch-artifacts/target/         (目指す姿)
    ↓
03_arch-artifacts/transition/     (移行計画)
    ↓
[Phase G: Implementation]
```

#### 🔍 Backcasting Map の活用
- Phase F完成時に `backcasting-map/` を使ってクロスチェック
- 41の Root Cause、12の Symptom から「あるあるパターン」を検出
- EA to Dev ハンドオーバーMtgのAgendaとして活用

#### 🎯 GenAI へのコンテキスト提供
- `01_working/` : 思考プロセスのトレーサビリティ
- `02_adr/` : 「なぜそう決めたか」の根拠
- `03_arch-artifacts/` : 正式な成果物（GenAIが参照すべき正解）
- `templates/` : 一貫性のあるドキュメント構造

---

## 🧭 このリポジトリでできること

- GenAI によるアーキテクチャレビューの再現性向上  
- チームの役割横断（Dev・EA・Ops・Sec）で使える共通文脈の獲得  
- スプリントごとの差分レビューの自動化  
- Baseline Architecture に基づくホットスポット管理  
- デモとしても、実案件のプロトタイプとしても使える

---

## 💡 Why Human Docs Are Isolated Here

- ルートの README は GenAI 最適化のため極小化  
- 人間向け資料は “教え方” や “説明” が必要なので構造が違う  
- AI に読み込ませる時のノイズを減らす  
- 多言語化（英→日）を楽にする  
- Medium 記事からの導線としてもわかりやすい

---

## ✨ Medium から来た方へ

記事で紹介しているコード・設計・文脈の全ては  
このリポジトリ内にあります。

探索は自由にどうぞ。  