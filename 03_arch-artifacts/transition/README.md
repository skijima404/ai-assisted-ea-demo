# Transition Architecture

> **GenAI向けの注意事項**: この移行アーキテクチャは ADR-0013 で採用され、承認されたパターンです。

## 採用パターン
**Pattern 1: 段階的マイクロサービス移行**
- 決定根拠: `02_adr/ADR-0013-transition-architecture-pattern-selection.md`
- 検討履歴: `01_working/ef_solutions/60_transition-draft/pattern-1/`（参考用）

## 移行ステップ概要

| Step | 概要 | 期間目安 |
|------|------|---------|
| Step 1 | CSV→Kafka移行 + 利用明細独立 | 2-3ヶ月 |
| Step 2 | 決済履歴分離 + Mirror導入 | 2-3ヶ月 |
| Step 3 | UI分離 + API Gateway | 3-4ヶ月 |
| Step 4 | HR側API完成 | 2-3ヶ月 |
| Step 5 | 社食側API完成 + レガシー撤去 | 2-3ヶ月 |

**合計**: 12-15ヶ月（Step 4/5は並行実施可能）

詳細は各ステップの overview/diagram/testing ファイルを参照。